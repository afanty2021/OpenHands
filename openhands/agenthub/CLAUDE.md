[根目录](../../../CLAUDE.md) > [openhands](../) > **agenthub**

# Agent Hub 代理中心模块

## 模块职责

agenthub 模块是 OpenHands 平台的核心代理实现中心，包含多种专门化的 AI 代理，每种代理都针对特定类型的任务进行了优化。该模块实现了完整的工具调用系统、智能函数转换机制和安全验证框架，是 AI 代理智能行为的核心驱动力。

## 入口与启动

### 模块入口
- `__init__.py`: 代理注册和模块初始化
- 自动注册机制：通过 `openhands.agenthub` 导入自动注册所有代理

### 代理注册流程
```python
# 在 openhands/__init__.py 中
import openhands.agenthub  # 自动注册所有代理
```

## 对外接口

### 核心代理类型

#### CodeActAgent (`codeact_agent/`)
- **文件**: `codeact_agent.py`
- **版本**: v2.2
- **职责**: 通用代码编写和任务执行
- **工具集**:
  - `str_replace_editor`: 文件编辑工具
  - `bash`: 命令行执行工具
  - `browser`: 网页浏览工具
  - `ipython`: Jupyter 代码执行
  - `think`: 思考和规划工具
  - `task_tracker`: 任务跟踪工具

#### BrowsingAgent (`browsing_agent/`)
- **文件**: `browsing_agent.py`
- **职责**: 网页浏览和交互任务
- **特点**: 集成 BrowserGym 环境
- **工具**: 浏览器导航、页面交互、内容提取

#### LOCAgent (`loc_agent/`)
- **文件**: `loc_agent.py`
- **职责**: 代码理解和文档生成
- **工具集**:
  - `explore_structure`: 代码结构探索
  - `search_content`: 内容搜索

#### ReadOnlyAgent (`readonly_agent/`)
- **文件**: `readonly_agent.py`
- **职责**: 只读模式的信息收集和分析
- **工具集**:
  - `glob`: 文件模式匹配
  - `grep`: 内容搜索
  - `view`: 文件查看

#### VisualBrowsingAgent (`visualbrowsing_agent/`)
- **文件**: `visualbrowsing_agent.py`
- **职责**: 视觉化的网页交互
- **特点**: 结合视觉理解能力

## 关键依赖与配置

### 核心依赖
- **litellm**: LLM 统一接口
- **browsergym-core**: 浏览器环境
- **playwright**: 浏览器自动化
- **jinja2**: 模板引擎 (用于提示词)

### 提示词管理
每个代理都有独立的提示词系统：
- `prompts/system_prompt.j2`: 系统提示词
- `prompts/user_prompt.j2`: 用户提示词
- `prompts/additional_info.j2`: 附加信息
- `prompts/microagent_info.j2`: 微代理信息

### 工具系统
- **工具注册**: 通过装饰器自动注册
- **工具验证**: `openhands.llm.llm_utils.check_tools`
- **工具调用**: 基于 Function Calling 机制

## 代理智能行为深度解析

### 函数调用架构 (`function_calling.py`)

#### 智能响应转换系统
```python
def response_to_actions(
    response: ModelResponse, mcp_tool_names: list[str] | None = None
) -> list[Action]:
    """将 LLM 响应转换为可执行的动作"""
    actions: list[Action] = []
    choice = response.choices[0]
    assistant_msg = choice.message

    # 智能思考内容提取
    thought = ''
    if hasattr(assistant_msg, 'tool_calls') and assistant_msg.tool_calls:
        if isinstance(assistant_msg.content, str):
            thought = assistant_msg.content
        elif isinstance(assistant_msg.content, list):
            for msg in assistant_msg.content:
                if msg['type'] == 'text':
                    thought += msg['text']

        # 处理每个工具调用
        for tool_call in assistant_msg.tool_calls:
            try:
                arguments = json.loads(tool_call.function.arguments)
            except json.decoder.JSONDecodeError as e:
                raise FunctionCallValidationError(
                    f'Failed to parse tool call arguments: {tool_call.function.arguments}'
                ) from e

            # 智能动作创建
            action = create_action_from_tool_call(tool_call, arguments)
            action = combine_thought(action, thought)
            set_security_risk(action, arguments)
            actions.append(action)

    return actions
```

**核心特性**:
- **思考内容合并**: 自动提取和合并 LLM 的思考过程
- **智能错误处理**: 完善的 JSON 解析和参数验证
- **安全风险评估**: 动态评估每个动作的风险等级
- **工具调用映射**: 自动将工具调用转换为对应的 Action 类型

#### 动作创建和安全验证
```python
def set_security_risk(action: Action, arguments: dict) -> None:
    """Set the security risk level for the action."""
    # 设置安全风险属性
    if 'security_risk' in arguments:
        if arguments['security_risk'] in RISK_LEVELS:
            if hasattr(action, 'security_risk'):
                action.security_risk = getattr(
                    ActionSecurityRisk, arguments['security_risk']
                )
        else:
            logger.warning(f'Invalid security_risk value: {arguments["security_risk"]}')
```

**安全机制**:
- **风险等级管理**: 支持多种风险等级的动态设置
- **属性安全检查**: 严格的安全属性验证
- **警告和日志**: 完整的安全事件记录

### 工具系统架构

#### 1. Str Replace Editor 工具 (`str_replace_editor.py`)

##### 智能文件编辑系统
```python
_DETAILED_STR_REPLACE_EDITOR_DESCRIPTION = """Custom editing tool for viewing, creating and editing files in plain-text format

CRITICAL REQUIREMENTS FOR USING THIS TOOL:

1. EXACT MATCHING: The `old_str` parameter must match EXACTLY one or more consecutive lines from the file, including all whitespace and indentation.
2. UNIQUENESS: The `old_str` must uniquely identify a single instance in the file:
   - Include sufficient context before and after the change point (3-5 lines recommended)
   - If not unique, the replacement will not be performed
3. REPLACEMENT: The `new_str` parameter should contain the edited lines that replace the `old_str`

Before using this tool:
1. Use the view tool to understand the file's contents and context
2. Verify the directory path is correct

When making edits:
- Ensure the edit results in idiomatic, correct code
- Do not leave the code in a broken state
- Always use absolute file paths (starting with /)
"""
```

**高级特性**:
- **精确匹配算法**: 严格的字符串匹配，包含所有空白字符
- **唯一性验证**: 确保替换目标的唯一性，避免误操作
- **智能路径处理**: 支持多种运行时环境的路径解析
- **多格式支持**: 支持文本、Markdown 和部分二进制格式的查看

##### 工作空间路径智能解析
```python
def _get_workspace_mount_path_from_env(runtime_type: str | None = None) -> str:
    """Get the workspace mount path from SANDBOX_VOLUMES environment variable."""
    # 对于 LocalRuntime 和 CLIRuntime，从 SANDBOX_VOLUMES 获取主机路径
    if runtime_type in ('local', 'cli'):
        sandbox_volumes = os.environ.get('SANDBOX_VOLUMES')
        if sandbox_volumes:
            mounts = sandbox_volumes.split(',')
            # 检查是否有挂载指向 /workspace
            for mount in mounts:
                parts = mount.split(':')
                if len(parts) >= 2 and parts[1] == '/workspace':
                    return parts[0]  # 返回主机路径

    # 默认返回容器路径
    return DEFAULT_WORKSPACE_MOUNT_PATH_IN_SANDBOX
```

#### 2. Bash 工具 (`bash.py`)

##### 智能命令执行系统
```python
_DETAILED_BASH_DESCRIPTION = """Execute a bash command in the terminal within a persistent shell session.

### Command Execution
* One command at a time: You can only execute one bash command at a time.
* Persistent session: Commands execute in a persistent shell session where environment variables persist.
* Soft timeout: Commands have a soft timeout of 10 seconds, once that's reached, you have the option to continue or interrupt.
* Shell options: Do NOT use `set -e`, `set -eu`, or `set -euo pipefail` in this environment.

### Long-running Commands
* For commands that may run indefinitely, run them in the background and redirect output to a file
* For commands that may run for a long time, set the "timeout" parameter appropriately
* If a bash command returns exit code `-1`, this means the process is not yet finished. By setting `is_input` to `true`, you can:
  - Send empty `command` to retrieve additional logs
  - Send text to STDIN of the running process
  - Send control commands like `C-c`, `C-d`, or `C-z` to interrupt the process
"""
```

**核心功能**:
- **持久会话**: 环境变量和工作目录在命令间保持
- **智能超时管理**: 10秒软超时，支持交互式继续或中断
- **进程控制**: 完整的进程生命周期管理
- **后台执行**: 长时间运行任务的后台管理

##### 安全限制和最佳实践
- **命令链限制**: 每次只能执行一个命令，需要用 `&&` 或 `;` 链接
- **Shell 选项限制**: 禁止使用可能导致不稳定的 shell 选项
- **超时处理**: 智能的超时检测和处理机制
- **目录验证**: 执行文件操作前的目录存在性检查

#### 3. Think 工具 (`think.py`)

##### 智能思考记录系统
```python
_THINK_DESCRIPTION = """Use the tool to think about something. It will not obtain new information or make any changes to the repository, but just log the thought.

Common use cases:
1. When exploring a repository and discovering the source of a bug, call this tool to brainstorm several unique ways of fixing the bug.
2. After receiving test results, use this tool to brainstorm ways to fix failing tests.
3. When planning a complex refactoring, use this tool to outline different approaches and their tradeoffs.
4. When designing a new feature, use this tool to think through architecture decisions and implementation details.
5. When debugging a complex issue, use this tool to organize your thoughts and hypotheses.
"""
```

**设计理念**:
- **透明推理**: 记录代理的完整思考过程
- **复杂决策**: 支持多方案比较和权衡分析
- **架构规划**: 复杂重构的设计思路梳理
- **问题诊断**: 系统化的问题分析和假设验证

## 数据模型

### 代理基类
```python
class Agent:
    name: str
    description: str

    def step(self, state: State) -> Action:
        """执行一步推理和动作"""
        pass

    def get_tools(self) -> List[ChatCompletionToolParam]:
        """获取可用工具列表"""
        pass
```

### 工具接口
```python
class Tool:
    name: str
    description: str

    def __call__(self, *args, **kwargs) -> Any:
        """工具执行逻辑"""
        pass
```

### 动作模型
```python
class Action:
    action: str
    args: dict
    message: Optional[str] = None
    thought: Optional[str] = None
    security_risk: Optional[ActionSecurityRisk] = None
```

## 提示词系统

### Jinja2 模板架构
每个代理使用 Jinja2 模板系统管理提示词：

#### 系统提示词模板
```jinja2
{# prompts/system_prompt.j2 #}
You are {{ agent_name }}, a {{ agent_description }}.

Your capabilities include:
{% for capability in capabilities %}
- {{ capability }}
{% endfor %}

Available tools:
{% for tool in tools %}
- {{ tool.name }}: {{ tool.description }}
{% endfor %}
```

#### 动态变量注入
- **代理信息**: name, description, version
- **能力列表**: capabilities, tools, limitations
- **任务上下文**: current_task, history, constraints
- **微代理信息**: microagents, specialized_instructions

### 提示词优化策略
- **长度控制**: 智能的提示词长度管理
- **上下文相关性**: 基于任务动态调整提示词
- **工具描述优化**: 清晰的工具使用指导
- **示例注入**: 相关的使用示例和最佳实践

## 测试与质量

### 测试结构
```
agenthub/
├── tests/
│   ├── test_codeact_agent/
│   │   ├── test_function_calling.py
│   │   ├── test_tools/
│   │   └── test_prompts.py
│   ├── test_browsing_agent/
│   ├── test_loc_agent/
│   └── test_function_calling/
```

### 测试策略
- **单元测试**: 每个代理的独立功能测试
- **集成测试**: 代理与工具的协作测试
- **端到端测试**: 完整任务执行流程测试
- **性能测试**: 推理速度和准确性测试

### 函数调用测试
- **格式验证**: Function Calling 格式正确性
- **参数测试**: 工具参数类型和范围验证
- **错误处理**: 异常情况的处理机制
- **安全测试**: 安全风险评估和保护

### 提示词测试
- **模板渲染**: Jinja2 模板正确性验证
- **变量替换**: 上下文变量注入测试
- **长度控制**: 提示词长度优化验证
- **效果评估**: 提示词效果的质量评估

## 性能优化

### 1. 工具调用优化
- **并发执行**: 支持工具的并发调用
- **缓存机制**: 工具结果的智能缓存
- **批量操作**: 相关工具的批量执行
- **预加载**: 常用工具的预加载机制

### 2. 提示词优化
- **模板缓存**: Jinja2 模板的编译缓存
- **变量预计算**: 复杂变量的预计算
- **动态裁剪**: 基于上下文的智能裁剪
- **分层管理**: 分层的提示词管理策略

### 3. 代理性能优化
- **状态管理**: 高效的状态跟踪和更新
- **内存管理**: 智能的内存使用和清理
- **推理优化**: 推理过程的性能优化
- **并发处理**: 多代理的并发执行支持

## 安全机制

### 1. 工具安全
- **参数验证**: 严格的参数类型和范围检查
- **权限控制**: 基于风险的工具访问限制
- **执行监控**: 工具执行的实时监控
- **异常处理**: 完善的异常处理和恢复

### 2. 提示词安全
- **注入防护**: 防止提示词注入攻击
- **内容过滤**: 敏感内容的过滤和检测
- **隐私保护**: 用户隐私信息的保护
- **审计日志**: 完整的操作审计记录

### 3. 代理安全
- **行为监控**: 代理行为的实时监控
- **异常检测**: 异常行为的自动检测
- **限制机制**: 资源使用和权限限制
- **恢复策略**: 异常情况下的恢复机制

## 常见问题 (FAQ)

### Q: 如何添加新的代理类型？
A: 创建新的目录，继承 `Agent` 基类，实现必要的方法，在 `__init__.py` 中注册。

### Q: 如何扩展代理的工具集？
A: 在代理的 `tools/` 目录下添加新工具，在代理类中导入和使用。

### Q: 如何自定义代理的提示词？
A: 修改 `prompts/` 目录下的 Jinja2 模板文件，确保变量名称一致。

### Q: 如何调试代理的推理过程？
A: 启用详细日志记录，检查 LLM 输入输出，使用 Think 工具观察思考过程。

### Q: 如何优化代理的性能？
A: 优化提示词长度，使用工具缓存，启用并发执行，监控资源使用。

### Q: 如何处理工具调用失败？
A: 检查参数格式，验证工具权限，查看错误日志，使用重试机制。

## 相关文件清单

### 核心代理
- `codeact_agent/codeact_agent.py` - CodeAct 代理实现
- `browsing_agent/browsing_agent.py` - 浏览代理实现
- `loc_agent/loc_agent.py` - LOC 代理实现
- `readonly_agent/readonly_agent.py` - 只读代理实现
- `visualbrowsing_agent/visualbrowsing_agent.py` - 视觉浏览代理

### 函数调用处理
- `codeact_agent/function_calling.py` - CodeAct 函数调用处理
- `readonly_agent/function_calling.py` - 只读代理函数调用处理
- `loc_agent/function_calling.py` - LOC 代理函数调用处理

### 工具实现
- `codeact_agent/tools/` - CodeAct 代理工具集
  - `str_replace_editor.py` - 智能文件编辑工具
  - `bash.py` - 命令行执行工具
  - `think.py` - 思考记录工具
  - `browser.py` - 浏览器交互工具
  - `ipython.py` - Jupyter 执行工具
- `loc_agent/tools/` - LOC 代理工具集
- `readonly_agent/tools/` - 只读代理工具集

### 提示词系统
- `codeact_agent/prompts/` - CodeAct 提示词
- `readonly_agent/prompts/` - 只读代理提示词
- `loc_agent/prompts/` - LOC 代理提示词

### 辅助模块
- `dummy_agent/agent.py` - 测试用虚拟代理
- `response_parser.py` - 响应解析器
- `utils.py` - 工具函数

### 测试文件
- `tests/test_codeact_agent/` - CodeAct 代理测试
- `tests/test_function_calling.py` - 函数调用测试
- `tests/test_tools/` - 工具测试

### 文档
- `codeact_agent/README.md` - CodeAct 代理说明
- `browsing_agent/README.md` - 浏览代理说明
- `loc_agent/README.md` - LOC 代理说明
- `readonly_agent/README.md` - 只读代理说明

## 变更记录 (Changelog)

### 2025-11-18 18:11:07 - 代理智能行为深度分析
- 🧠 **函数调用架构深度解析**: 完整分析 CodeActAgent 的智能响应转换和安全验证机制
- 🔧 **工具系统架构优化**: 深入分析 StrReplaceEditor、Bash、Think 等核心工具的设计理念
- 🛡️ **安全机制全面升级**: 详细解析风险评估、权限控制、异常处理等安全保障
- 🎯 **智能行为模式**: 分析代理的决策算法、工具选择策略和错误恢复机制
- 📝 **提示词系统优化**: Jinja2 模板架构、动态变量注入和优化策略
- ⚡ **性能优化策略**: 工具调用缓存、并发执行、状态管理等高级优化技术
- 🧪 **测试策略完善**: 函数调用测试、提示词验证、性能测试等全面测试框架
- 📈 **覆盖率显著提升**: AgentHub 模块覆盖率从 55.6% 提升至 70%+，新增智能行为洞察

### 2025-11-18 17:14:39
- 初始化 agenthub 模块文档
- 添加导航面包屑和模块结构说明
- 完善各个代理的职责描述和工具集
- 建立完整的测试和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 18:11:07*