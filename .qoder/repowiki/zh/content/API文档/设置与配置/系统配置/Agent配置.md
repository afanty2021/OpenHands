# Agent配置

<cite>
**本文档引用的文件**
- [agent_config.py](file://openhands/core/config/agent_config.py)
- [config.template.toml](file://config.template.toml)
- [security/README.md](file://openhands/security/README.md)
- [settings.ts](file://frontend/src/services/settings.ts)
- [settings.types.ts](file://frontend/src/settings-service/settings.types.ts)
</cite>

## 目录
1. [简介](#简介)
2. [代理类型配置](#代理类型配置)
3. [默认工具集](#默认工具集)
4. [任务规划策略与执行模式](#任务规划策略与执行模式)
5. [确认模式与安全分析器](#确认模式与安全分析器)
6. [代理记忆与上下文管理](#代理记忆与上下文管理)
7. [不同使用场景的配置建议](#不同使用场景的配置建议)
8. [配置继承机制与优先级规则](#配置继承机制与优先级规则)
9. [通过API动态调整代理配置](#通过api动态调整代理配置)

## 简介
本文档详细介绍了OpenHands AI代理的核心行为参数配置。涵盖了代理类型配置、默认工具集、任务规划策略、执行模式、确认模式、安全分析器、自动执行阈值、代理记忆和上下文管理等高级配置选项。同时提供了开发模式与生产模式的配置差异建议，以及配置继承机制和优先级规则的说明。

**Section sources**
- [agent_config.py](file://openhands/core/config/agent_config.py#L15-L158)
- [config.template.toml](file://config.template.toml#L231-L543)

## 代理类型配置
OpenHands支持多种代理类型，每种类型具有不同的功能和行为特征。代理类型通过`agent`配置项指定，可以在配置文件中定义不同代理的特定行为。

主要代理类型包括：
- **CodeActAgent**: 默认代理，支持代码执行、文件操作和命令行交互
- **BrowsingAgent**: 专注于网页浏览和信息检索
- **RepoExplorerAgent**: 用于代码库探索和分析
- **CustomAgent**: 自定义代理，可通过classpath指定自定义实现

代理配置支持继承机制，可以定义默认配置并为特定代理类型提供覆盖配置。例如，在`config.toml`中可以这样配置：

```toml
[agent]
enable_browsing = true
enable_jupyter = true

[agent.RepoExplorerAgent]
llm_config = 'gpt3'

[agent.CustomAgent]
classpath = "my_package.my_module.MyCustomAgent"
```

**Section sources**
- [agent_config.py](file://openhands/core/config/agent_config.py#L81-L158)
- [config.template.toml](file://config.template.toml#L237-L287)

## 默认工具集
OpenHands代理支持多种内置工具，这些工具可以通过配置启用或禁用。默认工具集包括：

- **浏览工具 (browsing)**: 启用网页浏览功能
- **Jupyter工具 (jupyter)**: 启用IPython/Jupyter执行环境
- **命令行工具 (cmd)**: 启用bash命令执行
- **编辑器工具 (editor)**: 启用文件编辑功能
- **思考工具 (think)**: 启用内部思考过程记录
- **完成工具 (finish)**: 启用任务完成确认
- **MCP工具 (mcp)**: 启用Model Context Protocol工具

工具的启用状态可以通过代理配置进行控制：

```python
class AgentConfig(BaseModel):
    enable_browsing: bool = Field(default=True)
    enable_jupyter: bool = Field(default=True)
    enable_cmd: bool = Field(default=True)
    enable_editor: bool = Field(default=True)
    enable_think: bool = Field(default=True)
    enable_finish: bool = Field(default=True)
    enable_mcp: bool = Field(default=True)
```

**Section sources**
- [agent_config.py](file://openhands/core/config/agent_config.py#L24-L44)
- [config.template.toml](file://config.template.toml#L239-L263)

## 任务规划策略与执行模式
OpenHands提供了灵活的任务规划和执行模式配置，支持复杂的任务分解和执行流程。

### 任务规划模式
通过`enable_plan_mode`参数控制是否启用长周期任务规划模式。当启用时，系统会自动使用长周期系统提示模板，并添加任务跟踪工具：

```python
@property
def resolved_system_prompt_filename(self) -> str:
    """
    根据代理配置返回适当的系统提示文件名。
    当enable_plan_mode为True时，自动使用长周期系统提示
    除非显式设置了自定义system_prompt_filename（不是默认值）。
    """
    if self.enable_plan_mode and self.system_prompt_filename == 'system_prompt.j2':
        return 'system_prompt_long_horizon.j2'
    return self.system_prompt_filename
```

### 任务跟踪工具
任务跟踪工具提供结构化的任务管理功能，支持以下操作：
- **view**: 查看当前任务列表
- **plan**: 创建或更新任务列表

任务状态包括：
- **todo**: 未开始
- **in_progress**: 进行中
- **done**: 已完成

**Section sources**
- [agent_config.py](file://openhands/core/config/agent_config.py#L52-L79)
- [task_tracker.py](file://openhands/agenthub/codeact_agent/tools/task_tracker.py#L103-L178)

## 确认模式与安全分析器
OpenHands提供了多层次的安全确认机制，确保代理操作的安全性和可控性。

### 确认模式
确认模式控制代理执行关键操作时是否需要用户确认。支持以下确认策略：
- **NeverConfirm**: 从不确认，代理自动执行所有操作
- **AlwaysConfirm**: 始终确认，所有操作都需要用户确认
- **ConfirmRisky**: 仅对高风险操作进行确认

```python
def test_toggle_confirmation_mode_transitions():
    """
    测试确认模式切换行为
    - 禁用 -> 启用：启用确认模式
    - 启用 -> 禁用：禁用确认模式
    """
```

### 安全分析器
安全分析器用于分析代理操作的安全风险，支持多种分析器类型：

- **LLM风险分析器 (默认)**: 利用LLM提供的风险评估
  - 使用LLM提供的风险评估（低、中、高）
  - 自动要求对高风险操作进行确认
  - 尊重确认模式设置
  - 轻量级，无外部依赖

- **Invariant分析器**: 使用外部分析工具检测潜在问题
  - 集成Invariant Analyzer进行工作流分析
  - 通过确认模式请求用户确认潜在风险操作

安全分析器配置示例：
```toml
[security]
confirmation_mode = false
security_analyzer = "llm"
```

**Section sources**
- [security/README.md](file://openhands/security/README.md#L1-L74)
- [agent_controller.py](file://openhands/controller/agent_controller.py#L197-L243)

## 代理记忆与上下文管理
OpenHands提供了强大的记忆和上下文管理功能，支持在长对话中保持上下文连贯性。

### 记忆压缩器 (Condenser)
记忆压缩器控制对话历史的管理和压缩方式，支持多种压缩策略：

- **noop**: 不压缩，保持完整历史（默认）
- **observation_masking**: 保持完整事件结构但屏蔽旧的观察结果
- **recent**: 只保留最近的事件，丢弃较旧的事件
- **llm**: 使用LLM总结对话历史
- **amortized**: 智能地遗忘旧事件同时保留重要上下文
- **llm_attention**: 使用LLM确定最相关的上下文

```toml
[condenser]
type = "noop"
# 或使用LLM总结
type = "llm"
llm_config = "condenser"
max_size = 100
```

### 上下文管理参数
- **enable_history_truncation**: 是否在达到LLM上下文长度限制时截断历史
- **condenser_max_size**: 压缩器最大大小
- **enable_default_condenser**: 是否启用默认的LLM总结压缩器

**Section sources**
- [config.template.toml](file://config.template.toml#L384-L435)
- [settings.ts](file://frontend/src/services/settings.ts#L16-L17)

## 不同使用场景的配置建议
根据不同的使用场景，建议采用不同的配置策略。

### 开发模式配置
开发模式注重灵活性和调试能力，建议配置：

```toml
[core]
debug = true
save_trajectory_path = "./trajectories"

[agent]
enable_think = true
enable_history_truncation = true

[security]
confirmation_mode = true
security_analyzer = "llm"
```

### 生产模式配置
生产模式注重稳定性和安全性，建议配置：

```toml
[core]
debug = false
max_budget_per_task = 10.0

[agent]
enable_browsing = false
enable_jupyter = false

[security]
confirmation_mode = true
security_analyzer = "invariant"
```

### 性能优化配置
对于资源受限环境的优化配置：

```toml
[condenser]
type = "recent"
max_events = 50

[agent]
enable_llm_editor = false
enable_prompt_extensions = false
```

**Section sources**
- [config.template.toml](file://config.template.toml#L9-L543)
- [settings.types.ts](file://frontend/src/settings-service/settings.types.ts#L1-L53)

## 配置继承机制与优先级规则
OpenHands采用分层配置继承机制，确保配置的灵活性和可维护性。

### 配置继承层次
1. **默认配置**: 系统内置的默认值
2. **全局配置**: `config.toml`中的全局设置
3. **代理类型配置**: 特定代理类型的配置
4. **运行时配置**: API调用时的动态配置

### 配置优先级规则
配置优先级从高到低：
1. 运行时API参数
2. 用户会话配置
3. 代理类型特定配置
4. 全局代理配置
5. 系统默认值

配置继承示例：
```python
@classmethod
def from_toml_section(cls, data: dict) -> dict[str, AgentConfig]:
    """
    从toml字典创建AgentConfig实例映射。
    默认配置从data中的非字典键构建。
    然后，每个具有字典值的键被视为自定义代理配置，
    其值覆盖默认配置。
    """
```

**Section sources**
- [agent_config.py](file://openhands/core/config/agent_config.py#L81-L158)
- [config.template.toml](file://config.template.toml#L237-L287)

## 通过API动态调整代理配置
OpenHands提供了API接口，支持在运行时动态调整代理配置。

### API配置端点
- **GET /api/settings**: 获取当前配置
- **POST /api/settings**: 保存配置更改
- **GET /api/security/settings**: 获取安全相关设置
- **POST /api/security/settings**: 更新安全设置

### 动态配置示例
```python
async def update_agent_config(agent_id: str, config_updates: dict):
    """
    动态更新指定代理的配置
    Args:
        agent_id: 代理标识符
        config_updates: 配置更新字典
    Returns:
        更新后的配置
    """
```

### MCP服务器配置
支持动态添加和管理MCP（Model Context Protocol）服务器：

```python
def useAddMcpServer():
    """
    添加MCP服务器的React Hook
    支持SSE、SHTTP和Stdio三种传输方式
    """
```

**Section sources**
- [settings.types.ts](file://frontend/src/settings-service/settings.types.ts#L1-L53)
- [settings-service.api.ts](file://frontend/src/settings-service/settings-service.api.ts#L1-L28)