# GitHub 认证配置

<cite>
**本文档中引用的文件**
- [constants.py](file://enterprise/server/auth/constants.py)
- [github_utils.py](file://enterprise/server/auth/github_utils.py)
- [token_manager.py](file://enterprise/server/auth/token_manager.py)
- [github_manager.py](file://enterprise/integrations/github/github_manager.py)
- [github_service.py](file://enterprise/integrations/github/github_service.py)
- [github.py](file://enterprise/server/routes/integration/github.py)
- [auth.py](file://enterprise/server/routes/auth.py)
- [github_proxy.py](file://enterprise/server/routes/github_proxy.py)
- [github_app_installation.py](file://enterprise/storage/github_app_installation.py)
- [generate-auth-url.ts](file://frontend/src/utils/generate-auth-url.ts)
- [use-github-auth-url.ts](file://frontend/src/hooks/use-github-auth-url.ts)
- [auth-service.api.ts](file://frontend/src/api/auth-service/auth-service.api.ts)
- [configure-github-repositories-anchor.tsx](file://frontend/src/components/features/settings/git-settings/configure-github-repositories-anchor.tsx)
</cite>

## 目录
1. [简介](#简介)
2. [系统架构概览](#系统架构概览)
3. [GitHub应用配置](#github应用配置)
4. [OAuth2认证流程](#oauth2认证流程)
5. [Webhook签名验证](#webhook签名验证)
6. [令牌管理](#令牌管理)
7. [权限控制与访问管理](#权限控制与访问管理)
8. [部署配置](#部署配置)
9. [安全最佳实践](#安全最佳实践)
10. [故障排除指南](#故障排除指南)
11. [总结](#总结)

## 简介

OpenHands提供了完整的GitHub OAuth2集成解决方案，支持企业级GitHub应用安装和Webhook事件处理。该系统采用多层认证架构，确保安全可靠的GitHub身份验证和API访问。

本文档详细介绍了GitHub认证配置的各个方面，包括应用注册、环境变量设置、OAuth流程实现、Webhook安全验证以及令牌生命周期管理。

## 系统架构概览

OpenHands的GitHub认证系统采用分层架构设计，包含前端认证界面、后端OAuth处理服务、数据库令牌存储和GitHub API交互层。

```mermaid
graph TB
subgraph "前端层"
A[用户界面] --> B[认证钩子]
B --> C[OAuth URL生成]
end
subgraph "后端服务层"
D[认证路由] --> E[Token管理器]
E --> F[GitHub服务]
F --> G[数据收集器]
end
subgraph "安全层"
H[Webhook验证] --> I[HMAC签名检查]
I --> J[令牌加密存储]
end
subgraph "外部服务"
K[GitHub API] --> L[GitHub应用]
M[Keycloak] --> N[身份提供商]
end
A --> D
D --> H
F --> K
E --> M
```

**图表来源**
- [auth.py](file://enterprise/server/routes/auth.py#L1-L50)
- [token_manager.py](file://enterprise/server/auth/token_manager.py#L1-L100)
- [github_manager.py](file://enterprise/integrations/github/github_manager.py#L1-L50)

## GitHub应用配置

### 应用注册与设置

在GitHub上注册OpenHands应用需要以下步骤：

1. **访问GitHub开发者设置**：进入个人或组织的GitHub开发者设置页面
2. **创建新应用**：选择"New GitHub App"并填写应用基本信息
3. **配置Webhook URL**：设置用于接收GitHub事件的Webhook端点
4. **选择权限范围**：配置所需的仓库和组织权限

### 关键配置参数

系统使用以下环境变量进行GitHub应用配置：

| 环境变量 | 描述 | 必需 | 示例值 |
|---------|------|------|--------|
| `GITHUB_APP_CLIENT_ID` | GitHub应用客户端ID | 是 | `ghp_xxxxxxxxxxxx` |
| `GITHUB_APP_CLIENT_SECRET` | GitHub应用客户端密钥 | 是 | `ghp_xxxxxxxxxxxx` |
| `GITHUB_APP_PRIVATE_KEY` | GitHub应用私钥（PEM格式） | 是 | `-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEA...\n-----END RSA PRIVATE KEY-----` |
| `GITHUB_APP_WEBHOOK_SECRET` | Webhook签名验证密钥 | 是 | `your-webhook-secret` |

**章节来源**
- [constants.py](file://enterprise/server/auth/constants.py#L3-L6)

### 应用权限配置

GitHub应用需要配置适当的权限范围以支持OpenHands功能：

- **仓库权限**：
  - 读取：issues、pull requests、contents、metadata
  - 写入：comments、statuses、deployments

- **组织权限**：
  - 读取：members、teams、organization projects

- **事件订阅**：
  - Issues
  - Pull Requests
  - Issue comments
  - Pull request reviews

## OAuth2认证流程

### 完整认证流程

OpenHands实现了标准的OAuth2授权码流程，支持代理认证和直接认证两种模式。

```mermaid
sequenceDiagram
participant U as 用户
participant F as 前端
participant B as 后端服务器
participant K as Keycloak
participant G as GitHub API
U->>F : 点击登录按钮
F->>B : 请求认证URL
B->>B : 生成状态参数
B->>U : 重定向到GitHub授权
U->>G : 授权应用访问权限
G->>B : 返回授权码
B->>K : 验证授权码
K->>B : 返回访问令牌
B->>B : 存储GitHub令牌
B->>U : 重定向回应用
```

**图表来源**
- [auth.py](file://enterprise/server/routes/auth.py#L99-L160)
- [github_proxy.py](file://enterprise/server/routes/github_proxy.py#L47-L70)

### 前端认证实现

前端通过React钩子实现OAuth认证流程：

```typescript
// 前端认证URL生成逻辑
const authUrl = useGitHubAuthUrl({
  appMode: config.APP_MODE,
  gitHubClientId: config.GITHUB_CLIENT_ID,
  authUrl: config.AUTH_URL
});
```

**章节来源**
- [use-github-auth-url.ts](file://frontend/src/hooks/use-github-auth-url.ts#L10-L15)
- [generate-auth-url.ts](file://frontend/src/utils/generate-auth-url.ts#L1-L37)

### 后端OAuth处理

后端服务器负责处理OAuth回调并管理令牌：

```python
# OAuth回调处理
@oauth_router.get('/keycloak/callback')
async def keycloak_callback(request: Request, code: Optional[str] = None):
    # 验证授权码
    keycloak_access_token, keycloak_refresh_token = await token_manager.get_keycloak_tokens(code, redirect_uri)
    
    # 获取用户信息
    user_info = await token_manager.get_user_info(keycloak_access_token)
    
    # 存储GitHub令牌
    await token_manager.store_idp_tokens(ProviderType.GITHUB, user_id, keycloak_access_token)
```

**章节来源**
- [auth.py](file://enterprise/server/routes/auth.py#L99-L160)

## Webhook签名验证

### HMAC-SHA256签名验证

GitHub Webhook事件的安全性依赖于HMAC-SHA256签名验证机制。

```mermaid
flowchart TD
A[接收Webhook请求] --> B[提取x-hub-signature-256头]
B --> C{签名是否存在?}
C --> |否| D[返回403错误]
C --> |是| E[计算预期签名]
E --> F[比较签名]
F --> G{签名匹配?}
G --> |否| H[返回403错误]
G --> |是| I[处理Webhook事件]
```

**图表来源**
- [github.py](file://enterprise/server/routes/integration/github.py#L26-L42)

### 签名验证实现

系统使用以下算法验证Webhook签名：

```python
def verify_github_signature(payload: bytes, signature: str):
    if not signature:
        raise HTTPException(status_code=403, detail='x-hub-signature-256 header is missing!')
    
    expected_signature = 'sha256=' + hmac.new(
        GITHUB_APP_WEBHOOK_SECRET.encode('utf-8'),
        msg=payload,
        digestmod=hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(expected_signature, signature):
        raise HTTPException(status_code=403, detail="Request signatures didn't match!")
```

**章节来源**
- [github.py](file://enterprise/server/routes/integration/github.py#L26-L42)

### Webhook事件处理

验证通过后，系统处理各种GitHub事件类型：

- **Issues事件**：新问题创建、评论、标签变更
- **Pull Request事件**：PR创建、更新、合并、关闭
- **Issue评论事件**：问题和PR的评论交互
- **Pull Request审查事件**：代码审查和批准

## 令牌管理

### 多层令牌存储架构

OpenHands实现了复杂的令牌管理系统，支持多种令牌类型和生命周期管理。

```mermaid
classDiagram
class TokenManager {
+get_idp_token(access_token, idp)
+store_idp_tokens(idp, user_id, keycloak_access_token)
+refresh_token(idp, refresh_token)
+store_org_token(installation_id, installation_token)
+load_org_token(installation_id)
}
class AuthTokenStore {
+store_tokens(access_token, refresh_token, expires_at)
+load_tokens(check_expiration)
}
class GithubAppInstallation {
+installation_id : String
+encrypted_token : String
+created_at : DateTime
+updated_at : DateTime
}
TokenManager --> AuthTokenStore
TokenManager --> GithubAppInstallation
```

**图表来源**
- [token_manager.py](file://enterprise/server/auth/token_manager.py#L78-L150)
- [github_app_installation.py](file://enterprise/storage/github_app_installation.py#L5-L23)

### GitHub App安装令牌

GitHub应用安装令牌通过专用表进行安全存储：

```python
class GithubAppInstallation(Base):
    id = Column(Integer, primary_key=True, autoincrement=True)
    installation_id = Column(String, nullable=False)
    encrypted_token = Column(String, nullable=False)
    created_at = Column(DateTime, server_default=text('CURRENT_TIMESTAMP'), nullable=False)
    updated_at = Column(DateTime, server_default=text('CURRENT_TIMESTAMP'), 
                       onupdate=text('CURRENT_TIMESTAMP'), nullable=False)
```

**章节来源**
- [github_app_installation.py](file://enterprise/storage/github_app_installation.py#L5-L23)

### 令牌刷新机制

系统实现了自动令牌刷新机制，确保长期访问的有效性：

```python
async def _refresh_github_token(self, refresh_token: str) -> dict[str, str | int]:
    url = 'https://github.com/login/oauth/access_token'
    payload = {
        'client_id': GITHUB_APP_CLIENT_ID,
        'client_secret': GITHUB_APP_CLIENT_SECRET,
        'refresh_token': refresh_token,
        'grant_type': 'refresh_token',
    }
    
    # 执行令牌刷新请求
    response = await client.post(url, data=payload)
    parsed = parse_qs(response.text)
    
    return await self._parse_refresh_response(parsed)
```

**章节来源**
- [token_manager.py](file://enterprise/server/auth/token_manager.py#L344-L368)

## 权限控制与访问管理

### 用户权限验证

系统实现了多层次的权限验证机制：

```mermaid
flowchart TD
A[用户请求] --> B{用户已认证?}
B --> |否| C[拒绝访问]
B --> |是| D{应用已安装?}
D --> |否| E[提示安装应用]
D --> |是| F{有写入权限?}
F --> |否| G[只读访问]
F --> |是| H[完全访问权限]
```

**图表来源**
- [github_manager.py](file://enterprise/integrations/github/github_manager.py#L121-L155)

### GitHub应用权限检查

系统检查用户对特定仓库的写入权限：

```python
def _user_has_write_access_to_repo(self, installation_id: str, full_repo_name: str, username: str) -> bool:
    with self.github_integration.get_github_for_installation(installation_id) as repos:
        repository = repos.get_repo(full_repo_name)
        
        # 检查协作者权限
        try:
            collaborator = repository.get_collaborator_permission(username)
            if collaborator in ['admin', 'write']:
                return True
        except Exception:
            pass
        
        # 检查组织成员权限
        org = repository.organization
        if org:
            user = org.get_members(username)
            return user is not None
            
        return False
```

**章节来源**
- [github_manager.py](file://enterprise/integrations/github/github_manager.py#L95-L119)

### 等待列表控制

系统支持基于Google Sheets或文本文件的用户等待列表控制：

```python
class UserVerifier:
    def is_user_allowed(self, username: str) -> bool:
        """基于文件和/或表格配置检查用户是否被允许"""
        if self.file_users:
            if username.lower() in self.file_users:
                return True
                
        if self.sheets_client and self.spreadsheet_id:
            sheet_users = [u.lower() for u in self.sheets_client.get_usernames(self.spreadsheet_id)]
            if username.lower() in sheet_users:
                return True
                
        return False
```

**章节来源**
- [github_utils.py](file://enterprise/server/auth/github_utils.py#L60-L79)

## 部署配置

### 环境变量配置

生产环境部署需要正确配置所有必要的环境变量：

```bash
# GitHub应用配置
export GITHUB_APP_CLIENT_ID="ghp_xxxxxxxxxxxx"
export GITHUB_APP_CLIENT_SECRET="ghp_xxxxxxxxxxxx"
export GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEA...\n-----END RSA PRIVATE KEY-----"
export GITHUB_APP_WEBHOOK_SECRET="your-webhook-secret"

# Keycloak配置
export KEYCLOAK_SERVER_URL="https://auth.example.com"
export KEYCLOAK_REALM_NAME="openhands"
export KEYCLOAK_CLIENT_ID="openhands-client"

# 应用配置
export AUTH_URL="auth.example.com"
export GITHUB_WEBHOOKS_ENABLED="true"
```

### 回调URL配置

正确的回调URL配置对于OAuth流程至关重要：

| 环境 | 回调URL格式 | 示例 |
|------|-------------|------|
| 开发环境 | `http://localhost:3000/oauth/keycloak/callback` | `http://localhost:3000/oauth/keycloak/callback` |
| 测试环境 | `https://staging.example.com/oauth/keycloak/callback` | `https://staging.example.com/oauth/keycloak/callback` |
| 生产环境 | `https://app.example.com/oauth/keycloak/callback` | `https://app.example.com/oauth/keycloak/callback` |

### GitHub应用安装配置

用户需要通过以下链接安装GitHub应用：

```typescript
const configureUrl = `https://github.com/apps/${slug}/installations/new`;
window.open(configureUrl, "_blank", "noreferrer noopener");
```

**章节来源**
- [configure-github-repositories-anchor.tsx](file://frontend/src/components/features/settings/git-settings/configure-github-repositories-anchor.tsx#L21-L25)

## 安全最佳实践

### 令牌安全存储

1. **加密存储**：所有令牌都使用JWT密钥进行加密存储
2. **访问控制**：严格的数据库访问权限控制
3. **定期轮换**：自动令牌刷新和轮换机制

### 网络安全配置

1. **HTTPS强制**：生产环境强制使用HTTPS
2. **CORS配置**：严格控制跨域请求
3. **IP白名单**：可选的IP地址限制

### API限制防护

1. **速率限制**：实现GitHub API速率限制检查
2. **重试机制**：指数退避的重试策略
3. **降级处理**：API不可用时的降级方案

### 审计日志

系统记录所有认证和授权活动：

```python
logger.info('user_logged_in', extra={
    'idp': idp,
    'idp_type': idp_type,
    'posthog_user_id': posthog_user_id,
    'is_feature_env': IS_FEATURE_ENV,
})
```

**章节来源**
- [auth.py](file://enterprise/server/routes/auth.py#L196-L204)

## 故障排除指南

### 常见认证问题

#### 1. 回调URL不匹配

**症状**：OAuth回调返回"invalid redirect_uri"错误

**解决方案**：
- 检查GitHub应用设置中的回调URL
- 确保与应用配置中的URL完全一致
- 验证协议（http vs https）和端口

#### 2. Webhook签名验证失败

**症状**：Webhook事件被拒绝，返回403错误

**解决方案**：
- 验证`GITHUB_APP_WEBHOOK_SECRET`环境变量
- 检查Webhook URL配置
- 确认签名算法正确（SHA256）

#### 3. 令牌获取失败

**症状**：无法获取GitHub访问令牌

**解决方案**：
- 验证GitHub应用客户端ID和密钥
- 检查应用权限设置
- 确认用户已正确授权应用

### 调试工具

#### 日志分析

启用详细日志记录进行问题诊断：

```python
logger.debug('Checking auth status for GitHub user')
logger.info('Got GitHub token from access token:', github_token)
```

#### 状态检查

使用健康检查端点验证系统状态：

```python
@app.post('/authenticate')
async def authenticate(request: Request):
    try:
        await get_access_token(request)
        return JSONResponse(status_code=200, content={'message': 'User authenticated'})
    except Exception:
        return JSONResponse(status_code=401, content={'error': 'User is not authenticated'})
```

**章节来源**
- [auth.py](file://enterprise/server/routes/auth.py#L295-L308)

### 性能优化

#### 缓存策略

- **令牌缓存**：短期缓存活跃用户的令牌
- **安装信息缓存**：缓存GitHub应用安装信息
- **用户权限缓存**：缓存用户仓库访问权限

#### 并发处理

- **异步处理**：使用异步IO处理Webhook事件
- **连接池**：复用HTTP连接减少延迟
- **批量操作**：批量处理多个仓库的操作

## 总结

OpenHands的GitHub认证配置系统提供了完整的企业级解决方案，涵盖了从应用注册到Webhook处理的整个OAuth2流程。系统采用多层安全架构，确保令牌安全存储和传输，同时提供灵活的权限控制和部署选项。

关键特性包括：
- 标准OAuth2流程支持
- 强大的Webhook签名验证
- 自动令牌管理和刷新
- 细粒度权限控制
- 企业级安全最佳实践
- 完善的故障排除机制

通过遵循本文档的配置指南和最佳实践，可以成功部署一个安全、可靠且高性能的GitHub认证系统。