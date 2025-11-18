[根目录](../../CLAUDE.md) > **openhands-cli**

# OpenHands CLI 命令行工具模块

## 模块职责

openhands-cli 模块提供 OpenHands 平台的命令行界面，为用户提供便捷的命令行工具来启动和管理 OpenHands 代理，支持交互式模式和脚本化执行。

## 入口与使用

### 主要入口
- CLI 脚本通过 pyproject.toml 定义的控制台脚本入口点
- 基于 Poetry 构建和打包

### 安装和使用
```bash
# 通过 pipx 安装（推荐）
pipx install openhands

# 通过 uv 运行
uvx --python 3.12 openhands

# 本地开发安装
pip install -e .
```

### 基本用法
```bash
# 启动交互式 CLI
openhands

# 启动 GUI 服务器
openhands serve

# 指定配置文件
openhands --config config.toml

# 使用特定微代理
openhands --microagent github

# 无头模式执行
openhands --headless
```

## 对外接口

### 命令行选项
- `--config, -c`: 指定配置文件路径
- `--microagent`: 选择特定的微代理
- `--headless`: 无头模式执行
- `--serve`: 启动 GUI 服务器
- `--port`: 指定服务器端口
- `--debug`: 启用调试模式
- `--version`: 显示版本信息
- `--help`: 显示帮助信息

### 配置管理
- **配置文件**: 支持 TOML 格式的配置文件
- **环境变量**: 通过环境变量覆盖配置
- **默认配置**: 合理的默认参数设置

### 交互功能
- **命令补全**: 支持命令自动补全
- **历史记录**: 命令历史保存和检索
- **彩色输出**: 增强的终端显示效果

## 关键依赖与配置

### 构建工具
- **Poetry**: Python 依赖管理和打包工具
- **PyInstaller**: 可执行文件构建
- **uv**: 快速 Python 包管理器

### CLI 框架
- **Click**: 现代 Python 命令行界面框架
- **Rich**: 丰富的终端显示库
- **prompt-toolkit**: 交互式命令行工具

### 打包配置
- **pyproject.toml**: Poetry 项目配置
- **openhands.spec**: PyInstaller 规格文件
- **build.py**: 自定义构建脚本

### 开发依赖
- **uv**: 用于依赖管理和虚拟环境
- **pytest**: 测试框架
- **mypy**: 类型检查工具

## 数据模型

### CLI 配置
```python
class CLIConfig:
    config_file: Optional[str]
    microagent: Optional[str]
    headless: bool
    serve: bool
    port: int
    debug: bool
    verbose: bool
```

### 命令选项
```python
class CommandOptions:
    command: str
    args: List[str]
    flags: Dict[str, bool]
    parameters: Dict[str, Any]
```

## 测试与质量

### 测试结构
```
openhands-cli/
├── tests/
│   ├── test_cli/
│   ├── test_commands/
│   └── test_integration/
```

### 测试策略
- **单元测试**: CLI 命令和选项解析测试
- **集成测试**: 与 OpenHands 核心的集成测试
- **端到端测试**: 完整的 CLI 工作流程测试
- **用户界面测试**: 交互式功能测试

### 质量保证
- **类型检查**: MyPy 静态类型检查
- **代码格式**: Black 和 isort 代码格式化
- **文档完整**: 详细的帮助文档和示例

### 性能优化
- **启动速度**: CLI 工具的快速启动
- **内存使用**: 最小化内存占用
- **响应性**: 交互式命令的快速响应

## 部署和分发

### 包管理
- **PyPI**: 通过 Python 包索引分发
- **pipx**: 推荐的安装方式
- **conda**: Conda 包支持（可选）

### 可执行文件
- **PyInstaller**: 生成独立可执行文件
- **跨平台**: 支持 Windows、macOS、Linux
- **依赖管理**: 处理运行时依赖

### 版本管理
- **语义化版本**: 遵循 SemVer 规范
- **自动发布**: CI/CD 自动发布流程
- **变更日志**: 详细的版本变更记录

## 常见问题 (FAQ)

### Q: 如何解决依赖冲突？
A: 使用虚拟环境或 pipx 隔离安装，确保依赖版本兼容性。

### Q: 如何自定义 CLI 命令？
A: 修改命令定义文件，添加新的命令和选项处理逻辑。

### Q: 如何调试 CLI 问题？
A: 使用 `--debug` 标志启用详细日志，检查配置文件和环境变量。

### Q: 如何在不同操作系统上使用？
A: CLI 工具支持跨平台，确保系统满足 Python 版本要求。

## 相关文件清单

### 构建配置
- `pyproject.toml` - Poetry 项目配置
- `openhands.spec` - PyInstaller 规格文件
- `build.py` - 自定义构建脚本
- `build.sh` - Shell 构建脚本
- `Makefile` - 构建和安装任务

### 依赖管理
- `uv.lock` - uv 依赖锁定文件

### 文档
- `README.md` - CLI 工具使用说明

### Python 模块
- `__init__.py` - 模块初始化（如果存在）

## 变更记录 (Changelog)

### 2025-11-18 17:14:39
- 初始化 openhands-cli 模块文档
- 添加导航面包屑和模块结构说明
- 完善 CLI 功能和使用方式描述
- 建立完整的测试和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:14:39*