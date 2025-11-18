[根目录](../../../CLAUDE.md) > [openhands](../) > **security**

# Security 安全分析模块

## 模块职责

security 模块是 OpenHands 平台的安全防护核心，提供多层次的安全分析和风险评估功能，通过集成多种安全引擎（Grayswan、Invariant、LLM-based 分析器）来检测和防范 AI 代理执行过程中的潜在安全风险。

## 入口与启动

### 主要入口文件
- `__init__.py`: 安全模块初始化和分析器注册
- `analyzer.py`: 安全分析器基类和接口定义
- `options.py`: 安全配置选项和设置

### 初始化流程
```python
# 安全系统通过 AgentController 集成
from openhands.security.analyzer import SecurityAnalyzer
from openhands.security.grayswan.analyzer import GrayswanAnalyzer
from openhands.security.invariant.analyzer import InvariantAnalyzer

# 创建安全分析器
security_analyzer = SecurityAnalyzer()
# 可选择性集成具体分析引擎
grayswan = GrayswanAnalyzer()
invariant = InvariantAnalyzer()
```

## 对外接口

### 核心安全接口
- **SecurityAnalyzer**: `analyzer.py` - 安全分析器基类
  - `security_risk(action: Action) -> ActionSecurityRisk`: 评估动作安全风险
  - `handle_api_request(request: Request) -> Any`: 处理 API 请求安全检查
  - `set_event_stream(event_stream) -> None`: 设置事件流访问权限

### 安全分析引擎
- **GrayswanAnalyzer**: `grayswan/analyzer.py` - Grayswan 安全引擎
  - 高级安全模式检测
  - 恶意行为模式识别
  - 复杂攻击场景分析

- **InvariantAnalyzer**: `invariant/analyzer.py` - Invariant 安全引擎
  - 不变式安全检查
  - 系统状态一致性验证
  - 策略违规检测

- **LLMSecurityAnalyzer**: `llm/analyzer.py` - LLM 安全分析器
  - 基于 LLM 的安全评估
  - 语义风险分析
  - 上下文安全检查

## 关键依赖与配置

### 核心依赖
- **fastapi**: Web 框架，用于 API 安全检查
- **pydantic**: 数据模型验证
- **openhands.events**: 事件系统和动作定义
- **第三方安全库**: Grayswan、Invariant SDK

### 安全配置
```python
# 安全配置示例
SECURITY_CONFIG = {
    "grayswan": {
        "enabled": True,
        "sensitivity": "medium",
        "policy_files": ["security/grayswan/policies/"]
    },
    "invariant": {
        "enabled": True,
        "check_interval": 1.0,
        "strict_mode": False
    },
    "llm_analyzer": {
        "enabled": True,
        "model": "gpt-4",
        "max_tokens": 1000
    }
}
```

## 数据模型

### 安全风险模型
```python
class ActionSecurityRisk(Enum):
    LOW = "low"           # 低风险，允许执行
    MEDIUM = "medium"     # 中等风险，需要确认
    HIGH = "high"         # 高风险，需要严格审查
    CRITICAL = "critical" # 严重风险，拒绝执行
    UNKNOWN = "unknown"   # 未知风险，保守处理
```

### 安全事件模型
```python
class SecurityEvent:
    action: Action
    risk_level: ActionSecurityRisk
    analysis_result: Dict[str, Any]
    timestamp: datetime
    analyzer: str
    reason: str
    recommendations: List[str]
```

### 安全策略模型
```python
class SecurityPolicy:
    name: str
    rules: List[SecurityRule]
    actions: List[str]  # 适用的动作类型
    risk_threshold: ActionSecurityRisk
    enabled: bool
```

## 安全分析策略

### 1. Grayswan 安全引擎
- **高级威胁检测**: 识别复杂的攻击模式和多步骤攻击
- **行为模式分析**: 基于历史行为的安全评估
- **实时威胁情报**: 集成最新的威胁情报数据库

### 2. Invariant 安全引擎
- **不变式检查**: 验证系统状态是否符合预定义的不变式
- **策略执行**: 强制执行安全策略和合规规则
- **状态监控**: 持续监控系统状态变化

### 3. LLM 安全分析器
- **语义理解**: 理解动作的语义和潜在影响
- **上下文评估**: 考虑执行上下文的安全影响
- **风险评估**: 综合评估多种安全风险因素

## 测试与质量

### 测试结构
```
security/
├── tests/
│   ├── test_analyzer.py
│   ├── test_grayswan.py
│   ├── test_invariant.py
│   ├── test_llm_analyzer.py
│   └── test_integration.py
```

### 测试策略
- **安全测试**: 各种攻击场景的安全检测测试
- **性能测试**: 安全分析的延迟和吞吐量测试
- **误报测试**: 评估安全引擎的准确性和误报率
- **集成测试**: 与 AgentController 的集成测试

### 质量指标
- **检测率**: 对已知威胁的检测成功率
- **误报率**: 正常动作被误判为威胁的比例
- **响应延迟**: 安全分析的平均处理时间
- **覆盖率**: 安全策略的覆盖范围

## 风险等级和处理策略

### LOW (低风险)
- **定义**: 常规操作，无明显安全风险
- **处理**: 直接允许执行
- **示例**: 文件读取、信息查询

### MEDIUM (中等风险)
- **定义**: 可能影响系统或数据安全的操作
- **处理**: 需要用户确认后执行
- **示例**: 文件修改、网络请求

### HIGH (高风险)
- **定义**: 可能造成重大安全影响的操作
- **处理**: 需要详细审查和用户授权
- **示例**: 系统配置修改、权限提升

### CRITICAL (严重风险)
- **定义**: 明显的安全威胁或攻击行为
- **处理**: 直接拒绝执行并记录安全事件
- **示例**: 恶意代码执行、系统破坏

## 常见问题 (FAQ)

### Q: 如何自定义安全策略？
A: 在 `invariant/policies/` 目录下添加策略文件，定义安全规则和不变式。

### Q: 安全分析是否会影响性能？
A: 安全分析是异步执行的，对主要操作流程的影响很小。高安全级别会增加一定的延迟。

### Q: 如何处理误报问题？
A: 可以调整安全引擎的敏感度设置，或添加白名单规则来减少误报。

### Q: 安全事件如何记录和报告？
A: 所有安全事件都会记录到日志中，并可以通过安全仪表板查看和分析。

### Q: 如何更新威胁情报？
A: Grayswan 引擎会自动更新威胁情报，也可以手动导入最新的安全策略文件。

## 相关文件清单

### 核心文件
- `__init__.py` - 安全模块初始化
- `analyzer.py` - 安全分析器基类
- `options.py` - 安全配置选项

### Grayswan 安全引擎
- `grayswan/__init__.py` - Grayswan 模块
- `grayswan/analyzer.py` - Grayswan 分析器
- `grayswan/utils.py` - 工具函数

### Invariant 安全引擎
- `invariant/__init__.py` - Invariant 模块
- `invariant/analyzer.py` - Invariant 分析器
- `invariant/client.py` - 客户端接口
- `invariant/nodes.py` - 节点定义
- `invariant/parser.py` - 策略解析器
- `invariant/policies.py` - 策略管理

### LLM 安全分析
- `llm/__init__.py` - LLM 安全模块
- `llm/analyzer.py` - LLM 安全分析器

### 配置和测试
- `tests/` - 测试文件
- 相关配置在 `openhands/core/config/` 中

## 变更记录 (Changelog)

### 2025-11-18 17:58:04 - 新增模块
- 完成安全分析模块深度扫描和分析
- 识别并分析三大安全引擎：Grayswan、Invariant、LLM
- 建立完整的安全风险分级和处理策略
- 添加安全配置和监控指南

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:58:04*