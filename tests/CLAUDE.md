[根目录](../CLAUDE.md) > **tests**

# Testing Infrastructure 测试基础设施

## 模块职责

tests 模块提供 OpenHands 平台的全面测试基础设施，包括端到端测试、集成测试、单元测试和性能测试。该模块确保系统各个组件的正确性、性能和可靠性，为持续集成和持续部署提供坚实的质量保证。

## 测试架构概览

### 测试层级结构
```
tests/
├── e2e/                    # 端到端测试
│   ├── test_conversation.py       # 对话流程测试
│   ├── test_settings.py           # 设置配置测试
│   ├── test_multi_conversation_resume.py  # 多对话恢复测试
│   └── test_browsing_catchphrase.py       # 浏览器交互测试
├── runtime/                # 运行时集成测试
│   ├── test_bash.py              # Bash 命令执行测试
│   ├── test_browsing.py          # 浏览器交互测试
│   ├── test_docker_images.py     # Docker 镜像测试
│   ├── test_replay.py            # 重放功能测试
│   └── test_microagent.py        # 微代理测试
├── unit/                   # 单元测试
│   ├── agenthub/              # 代理中心单元测试
│   ├── app_server/            # 应用服务器测试
│   ├── controller/            # 控制器测试
│   ├── core/                  # 核心功能测试
│   ├── events/                # 事件系统测试
│   ├── frontend/              # 前端组件测试
│   ├── integrations/          # 集成模块测试
│   ├── memory/                # 内存管理测试
│   ├── resolver/              # 问题解析器测试
│   └── security/              # 安全模块测试
└── integration/             # 集成测试（待扩展）
```

## 入口与执行

### 测试运行方式
```bash
# 运行所有测试
poetry run pytest ./tests

# 运行特定测试类型
poetry run pytest ./tests/unit          # 单元测试
poetry run pytest ./tests/runtime       # 运行时集成测试
poetry run pytest ./tests/e2e           # 端到端测试

# 运行特定测试文件
poetry run pytest ./tests/runtime/test_bash.py
poetry run pytest ./tests/unit/agenthub/test_agents.py

# 运行特定测试方法
poetry run pytest ./tests/runtime/test_bash.py::test_bash_command_env

# 详细输出模式
poetry run pytest -v ./tests/runtime/test_bash.py
poetry run pytest -vv ./tests/unit/test_llm_fncall_converter.py

# 并行运行（如果安装了 pytest-xdist）
poetry run pytest -n auto ./tests/unit
```

### 环境配置
```bash
# E2E 测试环境变量
export GITHUB_TOKEN="your-github-token"
export LLM_MODEL="gpt-4o"
export LLM_API_KEY="your-llm-api-key"
export LLM_BASE_URL="https://api.openai.com/v1"  # 可选

# 运行时测试环境变量
export TEST_RUNTIME="docker"           # docker, local, remote, runloop, daytona
export TEST_IN_CI="false"              # CI 环境标识
export RUN_AS_OPENHANDS="true"         # 用户权限设置
export SANDBOX_BASE_CONTAINER_IMAGE="custom-image"  # 自定义镜像
```

## 端到端测试 (E2E)

### 测试范围和功能
端到端测试使用 Playwright 自动化浏览器操作，验证 OpenHands 应用的完整用户流程。

#### GitHub Token 配置测试
- **文件**: `test_settings.py::test_github_token_configuration`
- **功能**: 验证 GitHub 令牌配置流程
- **流程**:
  1. 导航到 OpenHands 应用
  2. 检查 GitHub 令牌配置状态
  3. 如未配置，导航到设置页面进行配置
  4. 验证令牌保存和仓库选择可用性

#### 对话启动测试
- **文件**: `test_conversation.py::test_conversation_start`
- **功能**: 验证完整的对话启动流程
- **流程**:
  1. 导航到应用（假设 GitHub 令牌已配置）
  2. 选择 "openhands-agent/OpenHands" 仓库
  3. 点击启动按钮
  4. 等待对话界面加载
  5. 验证代理初始化
  6. 发送测试问题并验证响应

#### 多对话恢复测试
- **文件**: `test_multi_conversation_resume.py::test_multi_conversation_resume`
- **功能**: 验证对话历史恢复和连续性
- **流程**:
  1. 启动新对话并提问
  2. 提取对话 ID 并导航离开
  3. 通过对话列表恢复同一对话
  4. 验证历史记录完整性
  5. 发送后续问题验证上下文感知

#### 浏览器交互测试
- **文件**: `test_browsing_catchphrase.py`
- **功能**: 验证浏览器环境下的网页交互能力
- **测试内容**: 网站导航、信息检索、表单填写等

#### 本地运行时测试
- **文件**: `test_local_runtime.py::test_headless_mode_with_dummy_agent_no_browser`
- **功能**: 测试本地运行时环境和虚拟代理

### E2E 测试配置
```bash
# 基础配置
cd tests/e2e
poetry run pytest test_settings.py::test_github_token_configuration test_conversation.py::test_conversation_start -v

# 自定义基础 URL
poetry run pytest test_conversation.py::test_conversation_start -v --base-url=https://my-openhands-instance.com

# 可视化浏览器模式
poetry run pytest test_conversation.py::test_conversation_start -v --no-headless --slow-mo=50
```

## 运行时集成测试

### 测试目标和覆盖范围
运行时测试专注于验证 OpenHands 运行时环境中的工具交互和系统集成。

#### 核心工具测试
- **Bash 命令执行** (`test_bash.py`):
  - 环境变量处理
  - 命令执行和输出
  - 错误处理和异常情况
  - 权限和安全限制

- **文件操作** (`test_fileops.py`):
  - 文件读写操作
  - 目录遍历和搜索
  - 文件权限检查

- **浏览器交互** (`test_browsing.py`):
  - 网页导航和交互
  - 表单填写和提交
  - JavaScript 执行

#### 环境和配置测试
- **Docker 镜像** (`test_docker_images.py`):
  - 镜像构建和部署
  - 容器运行和管理
  - 资源限制和监控

- **环境变量** (`test_env_vars.py`):
  - 环境配置传递
  - 敏感信息处理
  - 配置验证

#### 高级功能测试
- **重放功能** (`test_replay.py`):
  - 会话录制和回放
  - 状态一致性验证
  - 错误恢复机制

- **微代理** (`test_microagent.py`):
  - 微代理加载和执行
  - 配置文件解析
  - 错误处理

- **IPython 集成** (`test_ipython.py`):
  - Jupyter 环境集成
  - 代码执行和输出
  - 交互式功能

#### 系统集成测试
- **MCP 动作** (`test_mcp_action.py`):
  - 模型控制协议集成
  - 动作执行和验证

- **LLM 编辑** (`test_llm_based_edit.py`):
  - AI 辅助代码编辑
  - 智能补全和修复

- **搜索工具** (`test_glob_and_grep.py`):
  - 文件搜索和内容查找
  - 正则表达式匹配

### 运行时测试配置
```bash
# 指定运行时环境
export TEST_RUNTIME="docker"    # 默认 Docker 运行时
export TEST_RUNTIME="local"     # 本地运行时
export TEST_RUNTIME="remote"    # 远程运行时

# 用户权限配置
export RUN_AS_OPENHANDS="true"  # 以 openhands 用户身份运行
export RUN_AS_OPENHANDS="false" # 以 root 用户身份运行

# 并发测试
poetry run pytest -n 4 ./tests/runtime
```

## 单元测试 (Unit)

### 测试组织结构

#### 代理中心测试 (`tests/unit/agenthub/`)
- **代理功能** (`test_agents.py`): 各种代理类型的单元测试
- **函数调用** (`test_function_calling.py`): 函数调用机制测试
- **提示缓存** (`test_prompt_caching.py`): 提示词缓存优化测试
- **工具集成** (`test_str_replace_editor_tool.py`): 编辑器工具测试
- **浏览器代理** (`test_browsing_agent/`): 浏览器专用代理测试

#### 应用服务器测试 (`tests/unit/app_server/`)
- **数据库会话** (`test_db_session_injector.py`): 会话管理测试
- **Docker 沙盒** (`test_docker_sandbox_service.py`): 沙盒环境测试
- **JWT 服务** (`test_jwt_service.py`): 认证令牌测试
- **API 客户端** (`test_httpx_client_injector.py`): HTTP 客户端测试
- **对话服务** (`test_sql_app_conversation_*_service.py`): 对话管理测试

#### 控制器测试 (`tests/unit/controller/`)
- **代理控制器** (`test_agent_controller.py`): 控制器核心功能
- **循环检测** (`test_is_stuck.py`): 无限循环检测算法
- **状态管理** (`test_agent_controller_loop_recovery.py`): 状态恢复机制
- **代理委托** (`test_agent_delegation.py`): 代理间委托机制

#### 核心功能测试 (`tests/unit/core/`)
- **配置管理** (`test_config*.py`): 全面的配置系统测试
- **日志记录** (`test_logging.py`, `test_logger.py`): 日志系统测试
- **消息处理** (`test_message_*.py`): 消息序列化和验证
- **事件系统** (`test_*.py`): 事件处理和序列化

#### 事件系统测试 (`tests/unit/events/`)
- **动作序列化** (`test_action_serialization.py`): 动作对象序列化
- **观察序列化** (`test_observation_serialization.py`): 观察对象序列化
- **事件流** (`test_event_stream.py`): 事件流处理
- **嵌套存储** (`test_nested_event_store.py`): 嵌套事件存储

#### 集成模块测试 (`tests/unit/integrations/`)
- **GitHub 集成** (`test_github_*.py`): GitHub API 集成测试
- **GitLab 集成** (`test_gitlab_*.py`): GitLab API 集成测试
- **Bitbucket 集成** (`test_bitbucket.py`): Bitbucket API 集成测试

### 测试工具和框架

#### 测试配置
```python
# conftest.py 示例配置
import pytest
import tempfile
import os
from pathlib import Path

@pytest.fixture
def temp_workspace():
    """提供临时工作空间目录"""
    with tempfile.TemporaryDirectory() as temp_dir:
        yield Path(temp_dir)

@pytest.fixture
def mock_config():
    """提供模拟配置"""
    return {
        "llm": {
            "model": "gpt-4",
            "api_key": "test-key",
            "temperature": 0.0
        },
        "sandbox": {
            "runtime": "docker",
            "base_image": "python:3.12-slim"
        }
    }
```

#### 测试工具
- **pytest**: 主要测试框架
- **pytest-asyncio**: 异步测试支持
- **pytest-mock**: Mock 和 patch 工具
- **pytest-cov**: 代码覆盖率报告
- **pytest-xdist**: 并行测试执行

## 质量保证和指标

### 测试覆盖率目标
- **单元测试覆盖率**: > 80%
- **集成测试覆盖率**: > 70%
- **关键路径覆盖率**: 100%

### 测试质量标准
- **测试独立性**: 每个测试应该独立运行
- **确定性**: 测试结果应该可重现
- **快速执行**: 单元测试应在秒级完成
- **清晰断言**: 明确的错误信息和断言

### 性能基准测试
- **响应时间**: API 响应时间基准
- **并发性能**: 多用户并发测试
- **内存使用**: 内存泄漏检测
- **资源清理**: 资源正确释放验证

## 持续集成集成

### GitHub Actions 集成
```yaml
# .github/workflows/test.yml 示例
name: Test Suite

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.11, 3.12]

    steps:
    - uses: actions/checkout@v4
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        pip install poetry
        poetry install

    - name: Run unit tests
      run: poetry run pytest ./tests/unit --cov=openhands

    - name: Run runtime tests
      run: poetry run pytest ./tests/runtime

    - name: Run E2E tests
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      run: |
        export GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }}
        export LLM_API_KEY=${{ secrets.LLM_API_KEY }}
        poetry run pytest ./tests/e2e
```

### 测试报告和监控
- **覆盖率报告**: Codecov 集成
- **测试结果**: GitHub Checks API
- **性能监控**: 测试执行时间跟踪
- **失败通知**: Slack/Email 通知

## 常见问题 (FAQ)

### Q: 如何调试失败的测试？
A: 使用 `-v` 详细输出模式，添加 `--pdb` 进入调试器，或使用 `pytest-sugar` 增强输出。

### Q: 如何运行特定的测试子集？
A: 使用 pytest 的标记系统 (`-m mark`) 或路径匹配 (`tests/unit/agenthub/`)。

### Q: 如何处理测试中的外部依赖？
A: 使用 Mock 和 Fixture 模拟外部依赖，或在 CI 环境中提供测试专用的服务。

### Q: 如何优化测试执行速度？
A: 使用并行执行 (`-n auto`)、智能测试选择 (`--lf`) 和缓存优化。

### Q: 如何测试异步代码？
A: 使用 `pytest-asyncio` 插件，添加 `@pytest.mark.asyncio` 装饰器。

## 相关文件清单

### 测试配置和工具
- `conftest.py` - pytest 配置和共享 fixtures
- `pytest.ini` - pytest 配置文件
- `requirements-test.txt` - 测试依赖（如果有）

### 端到端测试
- `e2e/conftest.py` - E2E 测试配置
- `e2e/test_conversation.py` - 对话流程测试
- `e2e/test_settings.py` - 设置配置测试
- `e2e/test_multi_conversation_resume.py` - 多对话恢复测试
- `e2e/test_browsing_catchphrase.py` - 浏览器交互测试
- `e2e/test_local_runtime.py` - 本地运行时测试

### 运行时集成测试
- `runtime/conftest.py` - 运行时测试配置
- `runtime/test_bash.py` - Bash 命令执行测试
- `runtime/test_browsing.py` - 浏览器交互测试
- `runtime/test_docker_images.py` - Docker 镜像测试
- `runtime/test_replay.py` - 重放功能测试
- `runtime/test_microagent.py` - 微代理测试

### 单元测试（主要模块）
- `unit/agenthub/` - 代理中心测试
- `unit/app_server/` - 应用服务器测试
- `unit/controller/` - 控制器测试
- `unit/core/` - 核心功能测试
- `unit/events/` - 事件系统测试
- `unit/integrations/` - 集成模块测试
- `unit/memory/` - 内存管理测试
- `unit/resolver/` - 问题解析器测试
- `unit/security/` - 安全模块测试

### 测试数据和工具
- `runtime/trajs/` - 测试轨迹数据
- `__init__.py` - 测试包初始化
- `README.md` - 测试模块说明

## 变更记录 (Changelog)

### 2025-11-18 18:34:32 - 测试基础设施建立
- **测试架构设计**：
  - 完整的三层测试架构（E2E、集成、单元）
  - Playwright 端到端测试框架集成
  - 多运行时环境支持（Docker、本地、远程）
  - 全面的测试配置和环境管理

- **E2E 测试体系**：
  - GitHub 令牌配置和认证流程测试
  - 完整的对话启动和管理测试
  - 多对话恢复和连续性验证
  - 浏览器交互和网页导航测试

- **运行时集成测试**：
  - Bash 命令执行和文件操作测试
  - Docker 容器管理和资源监控
  - 浏览器环境和网页交互测试
  - 重放功能和状态一致性验证

- **单元测试覆盖**：
  - 代理中心和应用服务器完整测试
  - 控制器和核心功能单元测试
  - 事件系统和消息处理测试
  - 第三方集成模块测试

### 2025-11-18 17:14:39
- 初始化 tests 模块文档
- 添加导航面包屑和模块结构说明
- 完善测试架构和执行指南
- 建立质量保证和CI/CD集成
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 18:34:32*