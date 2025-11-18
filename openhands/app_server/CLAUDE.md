[根目录](../../../CLAUDE.md) > [openhands](../) > **app_server**

# App Server 后端服务模块

## 模块职责

app_server 模块是 OpenHands 平台的后端服务层，基于 FastAPI 构建，负责处理用户会话、事件管理、沙盒控制和与前端实时通信。

## 入口与启动

### 主要入口文件
- `__init__.py`: 模块初始化和 FastAPI 应用创建
- `config.py`: 服务器配置管理
- `errors.py`: 全局错误处理

### 启动方式
通过 OpenHands 主程序启动，或直接运行 FastAPI 应用：
```bash
# 通过 OpenHands CLI 启动
uvx --python 3.12 openhands serve

# 直接运行（开发模式）
uvicorn openhands.app_server:app --reload
```

### 服务架构
- **FastAPI 应用**: 高性能异步 Web 框架
- **Socket.IO**: 实时双向通信
- **SQLAlchemy**: 数据库 ORM
- **Alembic**: 数据库迁移管理

## 对外接口

### 核心路由模块
- **会话管理**: `app_conversation/app_conversation_router.py`
  - POST `/conversations` - 创建新会话
  - GET `/conversations/{conversation_id}` - 获取会话信息
  - POST `/conversations/{conversation_id}/start` - 启动任务执行

- **事件处理**: `event/event_router.py`
  - GET `/events/{conversation_id}` - 获取事件流
  - WebSocket `/ws/{conversation_id}` - 实时事件推送

- **事件回调**: `event_callback/webhook_router.py`
  - POST `/webhook` - 处理外部事件回调

- **沙盒管理**: `sandbox/` 相关路由
  - 获取和管理沙盒规格
  - 沙盒状态监控

### 服务层接口
- **AppConversationService**: `app_conversation/app_conversation_service.py`
  - 会话生命周期管理
  - 任务执行控制

- **EventService**: `event/event_service.py`
  - 事件流管理
  - 事件过滤和分发

- **SandboxService**: `sandbox/` 各种沙盒服务
  - Docker 沙盒: `docker_sandbox_service.py`
  - 进程沙盒: `process_sandbox_service.py`
  - 远程沙盒: `remote_sandbox_service.py`

## 关键依赖与配置

### 核心框架依赖
- **FastAPI**: 现代化 Python Web 框架
- **uvicorn**: ASGI 服务器
- **python-socketio**: Socket.IO 服务器实现
- **sqlalchemy**: 数据库 ORM
- **alembic**: 数据库迁移工具

### 数据库和存储
- **PostgreSQL**: 主数据库 (通过 asyncpg/pg8000)
- **Redis**: 缓存和会话存储
- **SQLAlchemy**: 异步 ORM 操作

### 通信和协议
- **SSE (Server-Sent Events)**: `sse-starlette`
- **WebSocket**: Socket.IO 双向通信
- **HTTP/HTTPS**: RESTful API 接口

### 配置系统
- **环境变量**: 通过 `python-dotenv` 加载
- **配置模板**: `config.template.toml`
- **数据库配置**: Alembic 配置文件

## 数据模型

### 会话模型
```python
class AppConversationInfo:
    conversation_id: str
    status: ConversationStatus
    created_at: datetime
    updated_at: datetime
    agent_state: Optional[Dict]
    metadata: Dict[str, Any]
```

### 事件模型
```python
class Event:
    id: str
    conversation_id: str
    type: EventType
    content: Dict[str, Any]
    timestamp: datetime
    source: EventSource
```

### 沙盒模型
```python
class SandboxSpec:
    runtime_type: RuntimeType
    config: Dict[str, Any]
    resources: ResourceSpec
    security_policies: List[SecurityPolicy]
```

## 测试与质量

### 测试结构
```
app_server/
├── tests/
│   ├── test_app_conversation/
│   ├── test_event/
│   ├── test_sandbox/
│   └── test_integration/
```

### 测试策略
- **单元测试**: pytest + pytest-asyncio
- **集成测试**: API 端点测试
- **数据库测试**: 使用测试数据库和事务回滚
- **性能测试**: 负载测试和并发测试

### 数据库管理
- **Alembic 迁移**: `app_lifespan/alembic/`
  - 版本控制和迁移脚本
  - 数据库模式演进

- **SQL 查询优化**:
  - 异步查询处理
  - 连接池管理
  - 索引优化

### 错误处理和监控
- **全局异常处理**: `errors.py`
- **日志记录**: 结构化日志输出
- **性能监控**: 请求时间跟踪
- **健康检查**: 服务状态监控

## 常见问题 (FAQ)

### Q: 如何扩展新的会话类型？
A: 在 `app_conversation/` 下添加新的服务实现，继承基础服务类，实现特定的会话管理逻辑。

### Q: 如何配置不同的数据库？
A: 修改数据库连接字符串，运行 Alembic 迁移，确保环境变量正确配置。

### Q: 如何添加新的事件类型？
A: 在 `event/` 模块中定义新的事件类型，更新事件处理器和序列化逻辑。

### Q: 如何调试 Socket.IO 连接问题？
A: 检查 Socket.IO 服务器日志，确认客户端连接参数，验证防火墙和代理配置。

## 相关文件清单

### 核心文件
- `__init__.py` - FastAPI 应用创建和配置
- `config.py` - 服务器配置管理
- `errors.py` - 全局错误处理器

### 会话管理
- `app_conversation/app_conversation_service.py` - 会话服务实现
- `app_conversation/app_conversation_router.py` - 会话路由
- `app_conversation/app_conversation_start_task_service.py` - 任务启动服务
- `app_conversation/sql_app_conversation_start_task_service.py` - SQL 实现

### 生命周期管理
- `app_lifespan/app_lifespan_service.py` - 应用生命周期服务
- `app_lifespan/oss_app_lifespan_service.py` - OSS 实现
- `app_lifespan/alembic/` - 数据库迁移管理

### 事件系统
- `event/event_service.py` - 事件服务实现
- `event/event_router.py` - 事件路由
- `event/filesystem_event_service.py` - 文件系统事件
- `event_callback/` - 事件回调处理

### 沙盒管理
- `sandbox/docker_sandbox_service.py` - Docker 沙盒
- `sandbox/process_sandbox_service.py` - 进程沙盒
- `sandbox/remote_sandbox_service.py` - 远程沙盒
- `sandbox/*_sandbox_spec_service.py` - 各种沙盒规格服务

### 数据库迁移
- `app_lifespan/alembic.ini` - Alembic 配置
- `app_lifespan/alembic/env.py` - 迁移环境
- `app_lifespan/alembic/versions/` - 迁移版本

### 工具和配置
- `app_lifespan/alembic/script.py.mako` - 迁移模板
- `README.md` - 模块说明文档

## 变更记录 (Changelog)

### 2025-11-18 17:14:39
- 初始化 app_server 模块文档
- 添加导航面包屑和模块结构说明
- 完善入口文件、接口、依赖和配置信息
- 建立完整的测试和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:14:39*