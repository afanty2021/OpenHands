[根目录](../../../CLAUDE.md) > [openhands](../) > **resolver**

# Resolver 问题解决模块

## 模块职责

resolver 模块是 OpenHands 平台的自动化问题解决系统，专门处理 GitHub、GitLab、Bitbucket 等平台上的 Issues 和 Pull Requests，能够自动分析问题、生成解决方案并提交修复代码。

## 入口与启动

### 主要入口
- `issue_resolver.py`: 核心问题解决器
- `resolve_issue.py`: 命令行入口
- `send_pull_request.py`: PR 提交工具

### 启动方式
```bash
# 通过命令行解决单个问题
python -m openhands.resolver.resolve_issue --issue-url <URL>

# 批量处理
python -m openhands.resolver.issue_resolver --config config.yml
```

## 对外接口

### 平台接口 (`interfaces/`)
- **GitHub**: `github.py`
  - Issue 获取和分析
  - PR 创建和管理
  - 仓库操作和克隆

- **GitLab**: `gitlab.py`
  - GitLab API 集成
  - Merge Request 处理

- **Bitbucket**: `bitbucket.py`
  - Bitbucket API 支持
  - Pull Request 管理

### 问题处理流程
1. **Issue 分析**: 解析问题描述和上下文
2. **代码获取**: 克隆相关仓库
3. **解决方案生成**: 使用 AI 代理生成修复代码
4. **补丁应用**: 应用生成的代码变更
5. **PR 创建**: 提交 Pull Request

### 补丁系统 (`patching/`)
- **Patch 应用**: `apply.py`
- **补丁解析**: `patch.py`
- **代码片段**: `snippets.py`
- **异常处理**: `exceptions.py`

## 关键依赖与配置

### Git 操作
- **PyGithub**: GitHub API 客户端
- **python-gitlab**: GitLab API 客户端
- **GitPython**: Git 仓库操作

### 代码分析
- **whatthepatch**: 补丁文件解析
- **tree-sitter-python**: Python 代码解析
- **bashlex**: Bash 命令解析

### 文本处理
- **fuzzywuzzy**: 模糊字符串匹配
- **python-levenshtein**: 字符串相似度计算

### 配置系统
- **配置文件**: YAML 格式的 resolver 配置
- **环境变量**: API 密钥和认证信息
- **模板系统**: Jinja2 提示词模板

## 数据模型

### Issue 模型
```python
class Issue:
    number: int
    title: str
    body: str
    author: str
    state: str
    labels: List[str]
    assignees: List[str]
    repository: Repository
```

### PR 模型
```python
class PullRequest:
    number: int
    title: str
    body: str
    head_branch: str
    base_branch: str
    state: str
    mergeable: bool
```

### 补丁模型
```python
class Patch:
    file_path: str
    changes: List[Change]
    additions: int
    deletions: int
```

## 提示词系统 (`prompts/`)

### 问题解决提示词
- **基础解决**: `resolve/basic.jinja`
- **带测试的解决**: `resolve/basic-with-tests.jinja`
- **对话指导**: `resolve/basic-conversation-instructions.jinja`

### 成功判断提示词
- **Issue 成功检查**: `guess_success/issue-success-check.jinja`
- **PR 反馈检查**: `guess_success/pr-feedback-check.jinja`
- **PR 审查检查**: `guess_success/pr-review-check.jinja`

### 仓库指导 (`prompts/repo_instructions/`)
- 项目特定的指导文件
- 编码规范和约定
- 测试要求和流程

## 测试与质量

### 测试结构
```
resolver/
├── tests/
│   ├── test_issue_resolver/
│   ├── test_patch_applier/
│   ├── test_interfaces/
│   └── test_prompts/
```

### 测试策略
- **单元测试**: 各个组件的独立测试
- **集成测试**: 完整的问题解决流程测试
- **模拟测试**: 使用模拟的 GitHub 数据
- **端到端测试**: 真实仓库的测试（谨慎使用）

### 质量保证
- **代码审查**: 生成的代码质量检查
- **测试覆盖**: 自动生成的测试用例
- **补丁验证**: 补丁应用前验证

## 常见问题 (FAQ)

### Q: 如何配置 GitHub 访问权限？
A: 设置 `GITHUB_TOKEN` 环境变量，确保有足够权限访问目标仓库。

### Q: 如何添加新的平台支持？
A: 在 `interfaces/` 下创建新的平台模块，实现标准的接口方法。

### Q: 如何自定义解决方案生成逻辑？
A: 修改 `prompts/` 目录下的 Jinja2 模板，调整提示词内容。

### Q: 如何处理私有仓库？
A: 配置相应的访问令牌，确保 resolver 有权限访问私有仓库。

## 相关文件清单

### 核心文件
- `issue_resolver.py` - 核心问题解决器
- `resolve_issue.py` - 命令行入口
- `send_pull_request.py` - PR 提交工具
- `issue_handler_factory.py` - Issue 处理器工厂
- `utils.py` - 工具函数

### 平台接口
- `interfaces/github.py` - GitHub 接口
- `interfaces/gitlab.py` - GitLab 接口
- `interfaces/bitbucket.py` - Bitbucket 接口
- `interfaces/issue.py` - Issue 基类
- `interfaces/issue_definitions.py` - Issue 类型定义

### 补丁系统
- `patching/apply.py` - 补丁应用
- `patching/patch.py` - 补丁解析
- `patching/snippets.py` - 代码片段
- `patching/exceptions.py` - 补丁异常
- `patching/README.md` - 补丁系统说明

### 提示词系统
- `prompts/resolve/` - 问题解决提示词
- `prompts/guess_success/` - 成功判断提示词
- `prompts/repo_instructions/` - 仓库指导文件

### 工具和输出
- `resolver_output.py` - 解决结果输出
- `visualize_resolver_output.py` - 结果可视化
- `examples/openhands-resolver.yml` - 配置示例

### 文档
- `README.md` - 模块说明

## 变更记录 (Changelog)

### 2025-11-18 17:14:39
- 初始化 resolver 模块文档
- 添加导航面包屑和模块结构说明
- 完善平台接口和问题解决流程描述
- 建立完整的测试和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:14:39*