[根目录](../../../CLAUDE.md) > [openhands](../) > **memory**

# Memory 内存管理模块

## 模块职责

memory 模块是 OpenHands 平台的记忆系统核心，负责管理对话上下文、事件历史记录以及智能的对话压缩和优化，确保 AI 代理能够有效管理长期对话并保持在合理的上下文窗口内。

## 入口与启动

### 主要入口文件
- `memory.py`: 内存基类和接口定义
- `conversation_memory.py`: 对话内存管理实现
- `view.py`: 内存视图和抽象层
- `condenser/condenser.py`: 对话压缩系统核心

### 初始化流程
```python
# 内存系统通常通过 Controller 初始化
from openhands.memory.conversation_memory import ConversationMemory
from openhands.memory.memory import Memory

# 创建内存实例
memory = ConversationMemory()
# 可选择性配置压缩器
condenser = create_condenser(condenser_config)
memory.set_condenser(condenser)
```

## 对外接口

### 核心内存接口
- **Memory**: `memory.py` - 内存基类接口
  - `add_event(event: Event)`: 添加事件到内存
  - `get_events() -> List[Event]`: 获取所有事件
  - `reset()`: 重置内存状态

- **ConversationMemory**: `conversation_memory.py` - 对话内存实现
  - 管理完整的对话历史
  - 支持事件流操作
  - 集成压缩器功能

### 压缩器系统接口
- **Condenser**: `condenser/condenser.py` - 压缩器抽象基类
  - `condense(events: List[Event]) -> Condensation`: 执行压缩
  - `get_condensation_metadata() -> Dict`: 获取压缩元数据

### 压缩器实现
- **AmortizedForgettingCondenser**: `impl/amortized_forgetting_condenser.py`
  - 渐进式遗忘策略，根据重要性保留事件

- **LLMSummarizingCondenser**: `impl/llm_summarizing_condenser.py`
  - 使用 LLM 进行智能摘要压缩

- **ConversationWindowCondenser**: `impl/conversation_window_condenser.py`
  - 滑动窗口策略，保留最近的事件

- **LLMAttentionCondenser**: `impl/llm_attention_condenser.py`
  - 基于注意力机制的事件选择

## 关键依赖与配置

### 核心依赖
- **pydantic**: 数据模型验证和序列化
- **openhands.events**: 事件系统定义
- **openhans.core.config**: 配置管理
- **openhands.llm**: LLM 接口（用于基于 LLM 的压缩器）

### 配置系统
```python
# CondenserConfig 示例
class CondenserConfig(BaseModel):
    max_events: int = 50
    compression_threshold: float = 0.8
    strategy: str = "llm_summarizing"
    llm_config: Optional[LLMConfig] = None
```

## 数据模型

### 事件模型
```python
class Event:
    id: str
    type: EventType
    content: Dict[str, Any]
    timestamp: datetime
    source: EventSource
    importance: Optional[float] = None
```

### 压缩结果模型
```python
class Condensation(BaseModel):
    action: CondensationAction
    original_count: int
    compressed_count: int
    compression_ratio: float
    metadata: Dict[str, Any]
```

### 内存视图模型
```python
class View:
    events: List[Event]
    total_events: int
    compressed_events: int
    view_type: ViewType
```

## 测试与质量

### 测试结构
```
memory/
├── tests/
│   ├── test_conversation_memory.py
│   ├── test_condenser.py
│   ├── test_amortized_forgetting.py
│   ├── test_llm_summarizing.py
│   └── test_conversation_window.py
```

### 测试策略
- **单元测试**: 每种压缩器策略的独立功能测试
- **集成测试**: 内存与压缩器的协作测试
- **性能测试**: 压缩算法的时间和空间复杂度测试
- **质量评估**: 压缩后对话的连贯性评估

### 质量指标
- **压缩比**: 压缩前后事件数量比例
- **信息保留率**: 关键信息的保留程度
- **处理延迟**: 压缩操作的执行时间
- **内存使用**: 内存占用量优化

## 压缩策略详解

### 1. AmortizedForgettingCondenser
- **原理**: 基于事件重要性的渐进式遗忘
- **适用场景**: 长期对话，需要保留关键信息
- **优势**: 计算效率高，信息保留精准

### 2. LLMSummarizingCondenser
- **原理**: 使用 LLM 生成事件摘要
- **适用场景**: 复杂对话，需要语义理解
- **优势**: 语义保持好，上下文连贯性强

### 3. ConversationWindowCondenser
- **原理**: 保留固定时间窗口内的事件
- **适用场景**: 实时对话，关注最近上下文
- **优势**: 实现简单，性能稳定

### 4. PipelineCondenser
- **原理**: 组合多种压缩策略的流水线
- **适用场景**: 复杂场景，需要多级处理
- **优势**: 策灵活，可定制化程度高

## 常见问题 (FAQ)

### Q: 如何选择合适的压缩策略？
A: 根据对话特征选择：实时对话用滑动窗口，长期对话用渐进遗忘，复杂场景用 LLM 摘要。

### Q: 压缩过程是否会丢失重要信息？
A: 不同策略有不同的信息保留机制。LLM 摘要策略语义保留最好，渐进遗忘策略重要性保留最精准。

### Q: 如何自定义压缩策略？
A: 继承 `Condenser` 基类，实现 `condense` 方法，并在配置中注册新的压缩器。

### Q: 压缩对性能的影响如何？
A: 压缩是异步执行的，不会阻塞主流程。基于 LLM 的策略会有额外的推理延迟。

### Q: 如何监控压缩效果？
A: 通过压缩元数据监控压缩比、处理延迟等指标，定期评估压缩质量。

## 相关文件清单

### 核心文件
- `memory.py` - 内存基类和接口定义
- `conversation_memory.py` - 对话内存实现
- `view.py` - 内存视图抽象

### 压缩系统
- `condenser/condenser.py` - 压缩器基类
- `condenser/impl/` - 各种压缩策略实现
- `condenser/impl/pipeline.py` - 流水线压缩器

### 压缩策略实现
- `condenser/impl/amortized_forgetting_condenser.py` - 渐进式遗忘
- `condenser/impl/llm_summarizing_condenser.py` - LLM 摘要
- `condenser/impl/conversation_window_condenser.py` - 滑动窗口
- `condenser/impl/llm_attention_condenser.py` - 注意力机制
- `condenser/impl/no_op_condenser.py` - 空操作（用于测试）
- `condenser/impl/structured_summary_condenser.py` - 结构化摘要

### 配置和工具
- `condenser/__init__.py` - 压缩器注册和导出
- 相关配置文件在 `openhands/core/config/` 中

## 变更记录 (Changelog)

### 2025-11-18 17:58:04 - 新增模块
- 完成内存管理模块深度扫描和分析
- 识别并分析 8 种对话压缩策略
- 建立完整的内存系统文档结构
- 添加压缩策略选择指南和最佳实践

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:58:04*