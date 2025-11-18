[根目录](../../../CLAUDE.md) > [openhands](../) > **microagent**

# Microagent 微代理模块

## 模块职责

microagent 模块提供轻量级、专业化的微代理系统，支持基于 Markdown 配置的代理创建、管理和部署。它通过简化的配置方式让用户能够快速创建特定领域的专家代理，无需编写复杂的代码。

## 入口与启动

### 核心类定义
- `microagent.py`: BaseMicroagent 基类和微代理加载器
- `types.py`: 微代理类型定义和数据模型

### 微代理加载流程
```python
# 从 Markdown 文件加载微代理
microagent = BaseMicroagent.load(
    path="path/to/agent.md",
    microagent_dir=Path("./microagents")
)

# 直接从内容创建
microagent = BaseMicroagent.load(
    path="custom_agent",
    file_content=markdown_content
)
```

## 对外接口

### BaseMicroagent 核心类

#### 微代理类型支持
- **知识库代理** (`REPO_KNOWLEDGE`): 项目特定知识和指导
- **用户自定义代理** (`USER`): 用户创建的专用代理
- **第三方代理** (自动识别): 兼容 `.cursorrules`、`agents.md` 等

#### 第三方兼容性
自动识别和转换以下第三方配置文件：
- `.cursorrules`: Cursor 编辑器规则文件
- `agents.md` / `agent.md`: 通用代理配置文件
- `.openhands_instructions`: OpenHands 传统指令文件

#### 动态加载机制
- **路径解析**: 相对路径和绝对路径智能处理
- **内容解析**: Frontmatter 元数据和内容分离
- **类型推断**: 基于文件名和内容的自动类型推断
- **验证检查**: 配置格式和内容的完整性验证

### MicroagentMetadata 元数据系统

#### 元数据结构
```python
@dataclass
class MicroagentMetadata:
    name: str
    description: Optional[str] = None
    version: Optional[str] = None
    author: Optional[str] = None
    tags: Optional[List[str]] = None
    capabilities: Optional[List[str]] = None
    limitations: Optional[List[str]] = None
    usage_examples: Optional[List[str]] = None
```

#### 元数据验证
- **必需字段检查**: 确保关键字段存在
- **类型验证**: 验证数据类型和格式
- **内容完整性**: 检查描述和示例的完整性
- **版本兼容**: 版本号格式和兼容性检查

### 特化微代理类

#### RepoMicroagent
- **项目知识**: 项目特定的上下文和规则
- **开发指导**: 编码标准和最佳实践
- **工具使用**: 项目相关工具的使用指南
- **协作规范**: 团队协作和代码审查标准

#### UserMicroagent
- **个性化配置**: 用户偏好的代理行为
- **专用工具**: 用户特定领域的工具集成
- **工作流程**: 自定义的工作流程和步骤
- **性能调优**: 针对特定任务的性能优化

## 关键依赖与配置

### 核心依赖
- **frontmatter**: Markdown Frontmatter 解析
- **pydantic**: 数据模型验证和序列化
- **pathlib**: 路径操作和文件系统访问
- **typing**: 类型注解和泛型支持

### 配置文件格式
```markdown
---
name: "specialist-agent"
description: "专门处理特定任务的专家代理"
version: "1.0.0"
author: "Developer Name"
tags: ["specialist", "automation", "analysis"]
capabilities:
  - "代码分析和重构"
  - "性能优化建议"
  - "错误诊断和修复"
limitations:
  - "仅支持 Python 项目"
  - "需要明确的任务描述"
---

# 代理说明

这是一个专门处理特定任务的微代理...

## 使用指南

1. 提供清晰的代码上下文
2. 描述具体的问题和期望
3. 遵循建议的最佳实践

## 示例用法

```
请分析这个函数的性能瓶颈...
```
```

### 环境变量配置
- `MICROAGENT_DIR`: 微代理默认目录
- `MICROAGENT_CACHE_ENABLED`: 启用缓存机制
- `MICROAGENT_VALIDATION_STRICT`: 严格验证模式

## 数据模型

### 微代理类型枚举
```python
class MicroagentType(Enum):
    REPO_KNOWLEDGE = "repo_knowledge"
    USER = "user"
    # 可扩展的其他类型
```

### 输入元数据
```python
@dataclass
class InputMetadata:
    source: str
    timestamp: datetime
    context: Optional[Dict[str, Any]] = None
    user_preferences: Optional[Dict[str, str]] = None
```

### 加载选项
```python
@dataclass
class LoadOptions:
    validate_content: bool = True
    strict_mode: bool = False
    cache_enabled: bool = True
    custom_validators: Optional[List[Callable]] = None
```

## 测试与质量

### 测试结构
```
microagent/
├── tests/
│   ├── test_microagent.py - 基础功能测试
│   ├── test_microagent_utils.py - 工具函数测试
│   ├── test_user_microagents.py - 用户代理测试
│   └── test_microagent_no_header.py - 无头文件测试
```

### 测试覆盖重点
- **加载功能**: 各种格式和来源的微代理加载
- **验证逻辑**: Frontmatter 和内容的验证检查
- **第三方兼容**: 第三方配置文件的兼容性测试
- **错误处理**: 异常情况和错误恢复测试
- **性能测试**: 大量微代理的加载和缓存性能

### 质量保证措施
- **Schema 验证**: 严格的 Pydantic 模型验证
- **内容检查**: Markdown 格式和内容完整性
- **安全验证**: 路径遍历和注入攻击防护
- **性能监控**: 加载时间和内存使用监控

### 缓存和优化
- **LRU 缓存**: 最近使用微代理的内存缓存
- **延迟加载**: 按需加载微代理内容
- **增量更新**: 配置文件变更的增量检测
- **并行处理**: 多微代理的并行加载

## 高级特性

### 智能类型推断
- **文件扩展名**: 基于文件扩展名的类型推断
- **内容分析**: 基于 Frontmatter 的智能分类
- **目录结构**: 根据目录结构的类型判断
- **用户偏好**: 学习用户的使用偏好模式

### 动态验证系统
- **自定义验证器**: 支持用户定义的验证逻辑
- **规则引擎**: 基于规则的内容验证
- **模板检查**: 预定义模板的结构验证
- **交叉引用**: 微代理间依赖关系的检查

### 版本管理
- **版本控制**: 微代理配置的版本管理
- **向后兼容**: 旧版本配置的兼容处理
- **迁移工具**: 版本升级的自动化迁移
- **变更追踪**: 配置变更的历史记录

### 集成扩展
- **插件系统**: 支持第三方扩展和插件
- **API 接口**: RESTful API 微代理管理
- **Web 界面**: 图形化的微代理配置工具
- **CLI 工具**: 命令行的微代理管理工具

## 常见问题 (FAQ)

### Q: 如何创建自定义微代理类型？
A: 继承 `BaseMicroagent` 类，实现特定的加载和验证逻辑，注册新的 `MicroagentType`。

### Q: 微代理配置支持哪些 Markdown 特性？
A: 支持标准的 Markdown 语法，包括 Frontmatter、代码块、表格、列表等。

### Q: 如何处理微代理之间的依赖关系？
A: 使用 Frontmatter 中的 `dependencies` 字段声明依赖，系统会自动检查和加载。

### Q: 微代理缓存如何工作？
A: 系统使用 LRU 缓存机制，自动管理内存中的微代理实例，支持 TTL 和手动清理。

### Q: 如何验证微代理配置的正确性？
A: 使用内置的验证器，或实现自定义验证逻辑，在加载时进行严格验证。

### Q: 支持哪些第三方配置格式？
A: 目前支持 `.cursorrules`、`agents.md`、`.openhands_instructions` 等格式，可通过插件扩展。

## 相关文件清单

### 核心实现
- `microagent.py` - BaseMicroagent 核心实现
- `types.py` - 类型定义和数据模型

### 工具函数
- `utils/__init__.py` - 工具函数模块
- `utils/microagent_utils.py` - 微代理工具函数

### 测试文件
- `tests/test_microagent.py` - 基础功能测试
- `tests/test_microagent_utils.py` - 工具函数测试
- `tests/test_user_microagents.py` - 用户代理测试
- `tests/test_microagent_no_header.py` - 无头文件测试

### 异常定义
- `../core/exceptions.py` - 微代理相关异常定义

### 集成点
- `../events/observation/agent.py` - 微代理事件集成
- `../server/routes/conversation.py` - Web API 集成

### 配置示例
- `../microagents/` - 微代理配置示例目录
- `.cursorrules` - Cursor 编辑器兼容配置
- `agents.md` - 通用代理配置示例

## 变更记录 (Changelog)

### 2025-11-18 18:02:20 - 深度增量更新
- 🤖 **微代理架构深度解析**: 完整分析 BaseMicroagent 类和动态加载机制
- 📝 **第三方兼容性**: 深入分析 .cursorrules、agents.md 等格式兼容策略
- 🔍 **智能类型推断**: 详细解析文件类型识别和内容分析机制
- ✅ **验证和缓存系统**: 分析自定义验证器、LRU 缓存和性能优化
- 🧪 **测试基础设施**: 深度分析测试结构、质量保证和性能测试
- 🔧 **高级特性**: 版本管理、插件系统和集成扩展功能
- 📊 **数据模型详解**: 完整的元数据系统和配置格式规范
- ✨ **新增模块文档**: 创建 Microagent 模块的完整技术文档

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 18:02:20*