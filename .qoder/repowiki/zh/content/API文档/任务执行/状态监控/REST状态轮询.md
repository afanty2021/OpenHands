# REST状态轮询

<cite>
**本文档引用的文件**   
- [manage_conversations.py](file://openhands/server/routes/manage_conversations.py)
- [conversation_info.py](file://openhands/server/data_models/conversation_info.py)
- [v1-conversation-service.types.ts](file://frontend/src/api/conversation-service/v1-conversation-service.types.ts)
- [v1-conversation-service.api.ts](file://frontend/src/api/conversation-service/v1-conversation-service.api.ts)
- [use-user-conversation.ts](file://frontend/src/hooks/query/use-user-conversation.ts)
- [use-active-conversation.ts](file://frontend/src/hooks/query/use-active-conversation.ts)
- [conversation-status.ts](file://frontend/src/types/conversation-status.ts)
</cite>

## 目录
1. [简介](#简介)
2. [API端点详细说明](#api端点详细说明)
3. [响应数据结构](#响应数据结构)
4. [轮询实现示例](#轮询实现示例)
5. [轮询最佳实践](#轮询最佳实践)
6. [错误处理策略](#错误处理策略)
7. [状态码说明](#状态码说明)

## 简介
REST状态轮询是一种客户端定期向服务器查询任务或会话状态的模式。在OpenHands系统中，这种模式被用于监控对话会话的执行状态。客户端通过定期调用GET /conversations/{id} API端点来获取会话的最新状态信息，包括会话是否正在运行、已完成或遇到错误。这种轮询机制允许前端界面实时更新用户界面，显示任务的进度和状态变化。

## API端点详细说明

### HTTP请求方法
GET

### URL参数
- **conversation_id**: 会话的唯一标识符，作为URL路径参数传递。该ID用于指定要查询状态的特定会话。

### 请求头要求
轮询请求需要包含适当的认证信息。系统支持多种认证机制：

1. **Bearer Token认证**: 在Authorization请求头中提供Bearer令牌
   ```
   Authorization: Bearer <api_key>
   ```

2. **会话API密钥**: 对于V1会话，使用X-Session-API-Key请求头
   ```
   X-Session-API-Key: <session_api_key>
   ```

3. **Cookie认证**: 系统也支持通过cookie进行认证，特别是当用户通过Web界面登录时。

### 认证机制
系统实现了多层认证机制来确保会话访问的安全性：

- 首先尝试使用Authorization头中的API密钥进行认证
- 如果API密钥认证失败，则尝试使用cookie进行认证
- 对于企业版(SaaS)部署，还支持Keycloak集成的OAuth认证
- 每个会话都有访问控制，确保用户只能访问自己有权访问的会话

**Section sources**
- [manage_conversations.py](file://openhands/server/routes/manage_conversations.py#L433-L466)
- [v1-conversation-service.api.ts](file://frontend/src/api/conversation-service/v1-conversation-service.api.ts#L284-L293)

## 响应数据结构

### 主要字段定义
响应返回一个包含会话信息的JSON对象，主要字段包括：

| 字段 | 类型 | 描述 |
|------|------|------|
| conversation_id | string | 会话的唯一标识符 |
| status | string | 会话的当前状态（如"RUNNING"、"STOPPED"等） |
| title | string | 会话的标题或名称 |
| last_updated_at | string | 会话最后更新的时间戳 |
| created_at | string | 会话创建的时间戳 |
| selected_repository | string | 会话关联的代码仓库 |
| git_provider | string | 使用的Git服务提供商 |
| num_connections | integer | 当前连接到会话的客户端数量 |
| url | string | 会话的访问URL |
| session_api_key | string | 用于访问会话的API密钥 |

### 状态字段可能取值
会话状态(status)字段可以取以下值：

- **STARTING**: 会话正在初始化和启动过程中
- **RUNNING**: 会话正在执行中
- **STOPPED**: 会话已停止或完成
- **ARCHIVED**: 会话已被归档
- **ERROR**: 会话执行过程中遇到错误

### 进度信息
虽然响应中没有直接的"progress"字段，但系统通过以下方式间接提供进度信息：
- 通过last_updated_at字段的变化频率来判断会话是否活跃
- 通过num_connections字段了解有多少客户端连接到会话
- 通过status字段的状态变化来跟踪会话的生命周期

### 错误信息
当会话状态为"ERROR"时，系统会提供相关的错误信息：
- 详细的错误消息描述问题原因
- 可能包含错误代码或分类
- 提供可能的解决方案或重试建议

**Section sources**
- [conversation_info.py](file://openhands/server/data_models/conversation_info.py#L10-L31)
- [v1-conversation-service.types.ts](file://frontend/src/api/conversation-service/v1-conversation-service.types.ts#L76-L94)
- [conversation-status.ts](file://frontend/src/types/conversation-status.ts#L1-L6)

## 轮询实现示例

### curl命令示例
以下是一个使用curl命令实现轮询的示例：

```bash
#!/bin/bash

CONVERSATION_ID="your-conversation-id-here"
API_KEY="your-api-key-here"

# 轮询函数
poll_status() {
    while true; do
        echo "查询会话状态: $CONVERSATION_ID"
        
        # 发送GET请求获取会话状态
        response=$(curl -s -X GET \
            -H "Authorization: Bearer $API_KEY" \
            -H "Content-Type: application/json" \
            "http://localhost:3000/api/conversations/$CONVERSATION_ID")
        
        # 解析响应
        status=$(echo $response | jq -r '.status')
        title=$(echo $response | jq -r '.title')
        
        echo "状态: $status"
        echo "标题: $title"
        echo "完整响应: $response"
        echo "---"
        
        # 根据状态决定是否继续轮询
        if [ "$status" = "STOPPED" ] || [ "$status" = "ERROR" ]; then
            echo "会话结束，停止轮询"
            break
        fi
        
        # 等待5秒后再次轮询
        sleep 5
    done
}

# 执行轮询
poll_status
```

### 前端代码示例
以下是使用React和TypeScript实现的前端轮询代码示例：

```typescript
import { useEffect, useState } from 'react';
import axios from 'axios';

interface ConversationStatus {
  conversation_id: string;
  status: 'STARTING' | 'RUNNING' | 'STOPPED' | 'ARCHIVED' | 'ERROR';
  title: string;
  last_updated_at: string;
  num_connections: number;
}

const useConversationPolling = (conversationId: string, apiKey: string) => {
  const [status, setStatus] = useState<ConversationStatus | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [isPolling, setIsPolling] = useState(true);

  const pollStatus = async () => {
    try {
      const response = await axios.get<ConversationStatus>(
        `/api/conversations/${conversationId}`,
        {
          headers: {
            'Authorization': `Bearer ${apiKey}`,
            'Content-Type': 'application/json',
          },
        }
      );

      setStatus(response.data);
      setError(null);

      // 如果会话已停止或出错，停止轮询
      if (response.data.status === 'STOPPED' || response.data.status === 'ERROR') {
        setIsPolling(false);
      }
    } catch (err: any) {
      setError(err.message);
      console.error('轮询失败:', err);
      
      // 根据错误类型决定是否继续轮询
      if (err.response?.status === 404) {
        // 会话不存在，停止轮询
        setIsPolling(false);
      } else if (err.code === 'ECONNREFUSED' || err.code === 'ECONNRESET') {
        // 网络连接错误，可以继续重试
        console.log('网络错误，将继续重试...');
      }
    }
  };

  useEffect(() => {
    if (!conversationId || !isPolling) return;

    // 初始立即查询一次
    pollStatus();

    // 设置轮询间隔
    const intervalId = setInterval(pollStatus, 5000); // 每5秒轮询一次

    // 清理函数
    return () => {
      clearInterval(intervalId);
    };
  }, [conversationId, isPolling]);

  const stopPolling = () => {
    setIsPolling(false);
  };

  const startPolling = () => {
    setIsPolling(true);
  };

  return {
    status,
    error,
    isPolling,
    stopPolling,
    startPolling,
    refresh: pollStatus,
  };
};

// 使用示例
const ConversationMonitor = ({ conversationId }: { conversationId: string }) => {
  const { status, error, isPolling } = useConversationPolling(conversationId, 'your-api-key');

  if (error) {
    return <div>错误: {error}</div>;
  }

  if (!status) {
    return <div>加载中...</div>;
  }

  return (
    <div>
      <h3>会话状态监控</h3>
      <p>会话ID: {status.conversation_id}</p>
      <p>状态: {status.status}</p>
      <p>标题: {status.title}</p>
      <p>最后更新: {new Date(status.last_updated_at).toLocaleString()}</p>
      <p>连接数: {status.num_connections}</p>
      <p>轮询状态: {isPolling ? '进行中' : '已停止'}</p>
    </div>
  );
};
```

**Section sources**
- [v1-conversation-service.api.ts](file://frontend/src/api/conversation-service/v1-conversation-service.api.ts#L284-L293)
- [use-user-conversation.ts](file://frontend/src/hooks/query/use-user-conversation.ts#L19-L38)

## 轮询最佳实践

### 轮询间隔策略
选择合适的轮询间隔对于平衡用户体验和系统性能至关重要：

1. **动态轮询间隔**: 根据会话状态调整轮询频率
   - 当会话状态为"STARTING"时，使用较短的间隔（如2-3秒）
   - 当会话状态为"RUNNING"时，使用中等间隔（如5-10秒）
   - 当会话状态为"STOPPED"或"ERROR"时，停止轮询或使用较长间隔

2. **指数退避**: 在遇到错误时，逐渐增加轮询间隔
   ```typescript
   let retryCount = 0;
   const maxRetryInterval = 30000; // 最大30秒
   
   const getNextInterval = () => {
     // 指数退避，从1秒开始，每次加倍，最大30秒
     return Math.min(1000 * Math.pow(2, retryCount), maxRetryInterval);
   };
   ```

### 避免过度请求
为避免对服务器造成不必要的压力，请遵循以下最佳实践：

1. **合理设置轮询频率**: 避免过于频繁的请求（如每秒多次）
2. **使用条件请求**: 如果服务器支持，使用ETag或Last-Modified头进行条件请求
3. **客户端缓存**: 在客户端缓存最近的响应，避免重复处理相同数据
4. **批量查询**: 如果需要监控多个会话，考虑使用批量查询API而不是多个单独请求

### 性能优化建议
1. **连接复用**: 使用HTTP Keep-Alive保持连接，减少TCP握手开销
2. **压缩响应**: 启用Gzip压缩减少网络传输量
3. **错误处理**: 实现优雅的错误处理，避免在服务器故障时持续重试
4. **资源清理**: 确保在组件卸载或会话结束时清除轮询定时器

**Section sources**
- [use-active-conversation.ts](file://frontend/src/hooks/query/use-active-conversation.ts#L16-L23)
- [use-user-conversation.ts](file://frontend/src/hooks/query/use-user-conversation.ts#L35-L37)

## 错误处理策略

### 网络错误处理
网络错误是轮询过程中常见的问题，需要妥善处理：

1. **连接拒绝(ECONNREFUSED)**: 服务器不可达
   - 实现重试机制，使用指数退避策略
   - 显示友好的错误消息给用户
   - 可以暂时增加轮询间隔

2. **连接重置(ECONNRESET)**: 连接被意外中断
   - 自动重试请求
   - 记录错误日志用于诊断

3. **超时错误**: 请求超时
   - 设置合理的请求超时时间（如10秒）
   - 超时后自动重试
   - 避免在超时后立即重试，可以等待一段时间

### 服务器错误处理
不同HTTP状态码需要不同的处理策略：

1. **404 Not Found**: 会话不存在
   - 停止轮询，因为会话可能已被删除
   - 通知用户会话不存在
   - 提供创建新会话的选项

2. **401 Unauthorized**: 认证失败
   - 检查API密钥是否有效
   - 提示用户重新认证
   - 停止轮询直到认证问题解决

3. **429 Too Many Requests**: 请求过于频繁
   - 遵循服务器返回的Retry-After头
   - 显著增加轮询间隔
   - 可以暂时停止轮询

4. **500 Internal Server Error**: 服务器内部错误
   - 实现重试机制
   - 使用指数退避策略
   - 记录错误详情用于报告

### 客户端错误处理实现
以下是完整的错误处理实现示例：

```typescript
const handlePollingError = (error: any) => {
  // 记录错误
  console.error('轮询错误:', error);
  
  // 根据错误类型采取不同措施
  if (error.response) {
    // 服务器响应了错误状态码
    const status = error.response.status;
    
    switch (status) {
      case 404:
        // 会话不存在
        console.log('会话不存在，停止轮询');
        return { shouldContinue: false, message: '会话不存在' };
        
      case 401:
      case 403:
        // 认证问题
        console.log('认证失败，需要重新登录');
        return { 
          shouldContinue: false, 
          message: '认证失败，请重新登录' 
        };
        
      case 429:
        // 请求过于频繁
        const retryAfter = error.response.headers['retry-after'];
        const waitTime = retryAfter ? parseInt(retryAfter) * 1000 : 30000;
        console.log(`请求过于频繁，等待${waitTime/1000}秒后重试`);
        return { 
          shouldContinue: true, 
          waitTime,
          message: `请求过于频繁，请稍后再试` 
        };
        
      case 500:
      case 502:
      case 503:
      case 504:
        // 服务器错误
        const backoffTime = Math.min(1000 * Math.pow(2, retryCount), 60000);
        console.log(`服务器错误，${backoffTime/1000}秒后重试`);
        return { 
          shouldContinue: true, 
          waitTime: backoffTime,
          message: '服务器暂时不可用，正在重试' 
        };
        
      default:
        // 其他错误
        return { 
          shouldContinue: true, 
          waitTime: 5000,
          message: `请求失败: ${error.message}` 
        };
    }
  } else if (error.request) {
    // 请求已发出但没有收到响应
    if (error.code === 'ECONNABORTED') {
      // 超时
      return { 
        shouldContinue: true, 
        waitTime: 10000,
        message: '请求超时，正在重试' 
      };
    } else {
      // 其他网络问题
      return { 
        shouldContinue: true, 
        waitTime: 5000,
        message: '网络连接问题，正在重试' 
      };
    }
  } else {
    // 其他错误
    return { 
      shouldContinue: false, 
      message: `未知错误: ${error.message}` 
    };
  }
};
```

**Section sources**
- [http_client.py](file://openhands/integrations/protocols/http_client.py#L77-L99)
- [retrieve-axios-error-message.ts](file://frontend/src/utils/retrieve-axios-error-message.ts#L1-L26)

## 状态码说明

### 成功响应
- **200 OK**: 请求成功，返回会话状态信息
  - 响应体包含会话的完整状态信息
  - 这是最常见的成功响应

### 客户端错误
- **400 Bad Request**: 请求格式错误
  - 可能原因：conversation_id格式不正确
  - 解决方案：检查URL参数是否正确

- **401 Unauthorized**: 未授权访问
  - 可能原因：缺少或无效的认证信息
  - 解决方案：提供有效的API密钥或登录凭证

- **403 Forbidden**: 禁止访问
  - 可能原因：用户无权访问指定会话
  - 解决方案：检查用户权限或会话所有权

- **404 Not Found**: 资源未找到
  - 可能原因：指定的conversation_id不存在
  - 解决方案：验证会话ID是否正确，或检查会话是否已被删除

### 服务器错误
- **500 Internal Server Error**: 服务器内部错误
  - 可能原因：服务器处理请求时遇到未预期的错误
  - 解决方案：稍后重试，或联系系统管理员

- **502 Bad Gateway**: 网关错误
  - 可能原因：服务器作为网关或代理时，从上游服务器收到无效响应
  - 解决方案：检查后端服务状态

- **503 Service Unavailable**: 服务不可用
  - 可能原因：服务器暂时过载或维护中
  - 解决方案：稍后重试，通常服务器会提供Retry-After头

- **504 Gateway Timeout**: 网关超时
  - 可能原因：服务器作为网关或代理时，未能及时从上游服务器收到响应
  - 解决方案：增加超时时间或稍后重试

**Section sources**
- [manage_conversations.py](file://openhands/server/routes/manage_conversations.py#L280-L288)
- [http_client.py](file://openhands/integrations/protocols/http_client.py#L83-L94)