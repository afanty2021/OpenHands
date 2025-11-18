[根目录](../../CLAUDE.md) > **openhands**

# OpenHands 核心模块

## 模块职责

openhands 模块是整个 OpenHands 平台的核心，包含 AI 代理实现、控制器、运行时环境、事件系统和 LLM 接口等关键组件。

## 入口与启动

### 主要入口文件
- `openhands/core/main.py`: 核心主程序入口
- `openhands/__init__.py`: 模块初始化
- `openhands/version.py`: 版本信息管理

### 启动流程
1. 配置加载和初始化
2. 创建代理、控制器、运行时和内存组件
3. 启动事件循环和任务执行
4. 处理用户输入和代理交互

## 对外接口

### 核心代理接口
- **CodeActAgent**: `openhands/agenthub/codeact_agent/codeact_agent.py`
  - 通用代码编写和任务执行代理
  - 支持文件编辑、命令执行、浏览器操作等工具
  - 版本：v2.2

- **BrowsingAgent**: `openhands/agenthub/browsing_agent/browsing_agent.py`
  - 专门用于网页浏览和交互任务
  - 集成 BrowserGym 浏览器环境

- **LOCAgent**: `openhands/agenthub/loc_agent/loc_agent.py`
  - 代码理解和文档生成代理
  - 支持代码结构探索和内容搜索

- **VisualBrowsingAgent**: `openhands/agenthub/visualbrowsing_agent/visualbrowsing_agent.py`
  - 视觉化的网页交互代理

### 控制器接口
- **AgentController**: `openhands/controller/agent_controller.py`
  - 代理生命周期管理
  - 状态跟踪和错误处理
  - 卡点检测和恢复机制

### 运行时接口
- **DockerRuntime**: `openhands/runtime/impl/docker/docker_runtime.py`
- **LocalRuntime**: `openhands/runtime/impl/local/local_runtime.py`
- **KubernetesRuntime**: `openhands/runtime/impl/kubernetes/kubernetes_runtime.py`
- **RemoteRuntime**: `openhands/runtime/impl/remote/remote_runtime.py`

## 关键依赖与配置

### 核心依赖
- **litellm**: LLM 统一接口层
- **openai**: OpenAI API 客户端
- **docker**: Docker 容器管理
- **fastapi**: Web 框架
- **playwright**: 浏览器自动化
- **browsergym-core**: 浏览器交互环境

### 配置系统
- **OpenHandsConfig**: `openhands/core/config/config.py`
- **AgentConfig**: `openhands/core/config/agent_config.py`
- **LLMConfig**: `openhands/core/config/llm_config.py`

### 插件系统
- **AgentSkills**: `openhands/runtime/plugins/agent_skills/`
- **Jupyter**: `openhands/runtime/plugins/jupyter/`
- **VSCode**: `openhands/runtime/plugins/vscode/`

## 数据模型

### 事件系统
- **Action**: `openhands/events/action/` - 代理动作事件
- **Observation**: `openhands/events/observation/` - 环境观察事件
- **Event**: `openhands/events/event.py` - 基础事件类
- **EventStream**: `openhands/events/stream.py` - 事件流管理

### 状态管理
- **State**: `openhands/controller/state/state.py` - 代理状态
- **StateTracker**: `openhands/controller/state/state_tracker.py` - 状态跟踪器

### 内存系统
- **ConversationMemory**: `openhands/memory/conversation_memory.py`
- **Condenser**: `openhands/memory/condenser/` - 对话压缩和总结

## 测试与质量

### 测试目录结构
```
openhands/
├── tests/
│   ├── test_agenthub/
│   ├── test_controller/
│   ├── test_runtime/
│   ├── test_events/
│   └── test_llm/
```

### 测试框架
- **pytest**: 主要测试框架
- **pytest-asyncio**: 异步测试支持
- **pytest-mock**: Mock 对象支持

### 质量工具
- **ruff**: 代码格式化和 lint
- **mypy**: 静态类型检查
- **pytest-cov**: 测试覆盖率

### 测试策略
- 单元测试：每个模块的独立功能测试
- 集成测试：模块间协作测试
- 端到端测试：完整任务执行流程测试
- 性能测试：内存和执行性能监控

## 常见问题 (FAQ)

### Q: 如何添加新的代理类型？
A: 在 `openhands/agenthub/` 下创建新目录，继承 `Agent` 基类，实现必要的方法和工具。

### Q: 如何扩展运行时环境？
A: 在 `openhands/runtime/impl/` 下创建新的运行时实现，继承 `Runtime` 基类。

### Q: 如何添加新的事件类型？
A: 在 `openhands/events/action/` 或 `openhands/events/observation/` 下创建新的事件类。

### Q: 如何配置不同的 LLM 提供商？
A: 修改 `openhands/llm/llm_registry.py` 注册新的 LLM 实现，并通过配置文件指定模型参数。

## 相关文件清单

### 核心文件
- `openhands/core/main.py` - 主程序入口
- `openhands/core/config/` - 配置系统
- `openhands/controller/agent_controller.py` - 代理控制器
- `openhands/controller/agent.py` - 代理基类

### 代理实现
- `openhands/agenthub/codeact_agent/` - CodeAct 代理
- `openhands/agenthub/browsing_agent/` - 浏览代理
- `openhands/agenthub/loc_agent/` - LOC 代理
- `openhands/agenthub/readonly_agent/` - 只读代理

### 运行时实现
- `openhands/runtime/base.py` - 运行时基类
- `openhands/runtime/impl/` - 各种运行时实现
- `openhands/runtime/plugins/` - 插件系统

### 事件系统
- `openhands/events/event.py` - 事件基类
- `openhands/events/stream.py` - 事件流
- `openhands/events/action/` - 动作事件
- `openhands/events/observation/` - 观察事件

### LLM 接口
- `openhands/llm/llm.py` - LLM 基类
- `openhands/llm/llm_registry.py` - LLM 注册表
- `openhands/llm/router/` - 路由系统

### 内存和对话
- `openhands/memory/memory.py` - 内存基类
- `openhands/memory/conversation_memory.py` - 对话内存
- `openhands/memory/condenser/` - 对话压缩

## 变更记录 (Changelog)

### 2025-11-18 17:14:39
- 初始化 openhands 核心模块文档
- 添加导航面包屑和模块结构说明
- 完善入口文件、接口、依赖和配置信息
- 建立完整的测试和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:14:39*