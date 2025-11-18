[根目录](../../CLAUDE.md) > **microagents**

# Microagents 微代理模块

## 模块职责

microagents 模块包含 OpenHands 平台的微代理定义和配置文件。微代理是专门化的 AI 代理配置，针对特定任务类型、平台或工作流程进行了优化，提供开箱即用的解决方案。

## 入口与使用

### 主要内容
- **代理定义文件**: Markdown 格式的微代理配置
- **技能配置**: 专门的工具和功能配置
- **使用指南**: 每个微代理的详细说明

### 使用方式
```bash
# 启动特定微代理
openhands --microagent github

# 列出可用微代理
openhands --list-microagents
```

## 对外接口

### 核心微代理类型

#### 开发相关代理
- **GitHub 代理**: `github.md`
  - GitHub Issue 和 PR 处理
  - 代码审查和协作
  - 仓库管理和维护

- **GitLab 代理**: `gitlab.md`
  - GitLab Merge Request 处理
  - CI/CD 管道集成
  - 项目管理功能

- **代码审查代理**: `code-review.md`
  - 自动化代码审查
  - 质量检查和反馈
  - 最佳实践建议

#### DevOps 代理
- **Docker 代理**: `docker.md`
  - 容器化应用开发
  - Dockerfile 生成和优化
  - 容器编排配置

- **Kubernetes 代理**: `kubernetes.md`
  - K8s 资源配置
  - 部署和服务管理
  - 集群运维任务

- **NPM 代理**: `npm.md`
  - JavaScript/Node.js 项目管理
  - 包管理和构建流程
  - 前端开发工具

#### 安全代理
- **安全代理**: `security.md`
  - 安全漏洞扫描和修复
  - 代码安全最佳实践
  - 依赖安全检查

#### 专业化代理
- **SSH 代理**: `ssh.md`
  - 远程服务器管理
  - 部署和运维任务
  - 网络和安全配置

- **LaTeX 代理**: `pdflatex.md`
  - 学术文档编写
  - LaTeX 排版和格式
  - 论文和报告生成

## 关键依赖与配置

### 代理配置格式
- **Markdown**: 易于阅读和编辑的配置格式
- **YAML 嵌入**: 结构化配置数据
- **模板系统**: 动态参数替换

### 工具和功能
- **专业工具集**: 针对特定领域的工具配置
- **工作流程**: 标准化的任务执行流程
- **最佳实践**: 领域专家的经验总结

### 平台集成
- **Git 平台**: GitHub, GitLab, Bitbucket 集成
- **云平台**: AWS, Azure, GCP 支持
- **开发工具**: IDE 和编辑器集成

## 数据模型

### 微代理配置结构
```markdown
# 代理名称

## 描述
简要描述代理的功能和用途

## 工具
- tool1: 工具描述
- tool2: 工具描述

## 使用场景
1. 场景一描述
2. 场景二描述

## 配置参数
- parameter1: 参数说明
- parameter2: 参数说明
```

### 代理元数据
- **名称和描述**: 代理的身份标识
- **工具列表**: 可用工具和功能
- **适用场景**: 推荐使用情况
- **配置选项**: 自定义参数设置

## 测试与质量

### 验证流程
- **配置验证**: 检查代理配置格式和内容
- **功能测试**: 验证工具和工作流程
- **集成测试**: 测试与平台的集成

### 质量保证
- **文档完整性**: 每个代理都有详细说明
- **示例和教程**: 提供使用示例和教程
- **社区反馈**: 收集和整合用户反馈

## 常见问题 (FAQ)

### Q: 如何创建新的微代理？
A: 复制现有代理模板，修改工具配置和使用场景描述。

### Q: 如何自定义现有微代理？
A: 编辑对应的 Markdown 配置文件，调整工具和参数设置。

### Q: 微代理与普通代理有什么区别？
A: 微代理是预配置的专门化代理，提供特定领域的最佳实践和工具组合。

### Q: 如何分享自定义微代理？
A: 将配置文件提交到 microagents 目录，或通过社区渠道分享。

## 相关文件清单

### 开发代理
- `github.md` - GitHub 开发代理
- `gitlab.md` - GitLab 开发代理
- `bitbucket.md` - Bitbucket 开发代理
- `code-review.md` - 代码审查代理
- `codereview-roasted.md` - 严格代码审查代理

### DevOps 代理
- `docker.md` - Docker 容器代理
- `kubernetes.md` - K8s 编排代理
- `npm.md` - NPM 包管理代理

### 安全代理
- `security.md` - 安全检查代理

### 工具代理
- `ssh.md` - SSH 远程管理代理
- `pdflatex.md` - LaTeX 文档代理

### 通用代理
- `default-tools.md` - 默认工具配置
- `onboarding.md` - 新用户入门代理

### 专用代理
- `agent-memory.md` - 记忆管理代理
- `browsing.md` - 浏览专用代理
- `swift-linux.md` - Swift Linux 开发代理

### 任务特定代理
- `add_agent.md` - 添加代理任务
- `add_repo_inst.md` - 添加仓库指令
- `address_pr_comments.md` - 处理 PR 评论
- `fix-py-line-too-long.md` - 修复 Python 行长
- `fix_test.md` - 测试修复
- `update_pr_description.md` - 更新 PR 描述
- `update_test.md` - 更新测试

### 文档
- `README.md` - 模块说明和使用指南
- `agent-builder.md` - 代理构建指南

## 变更记录 (Changelog)

### 2025-11-18 17:14:39
- 初始化 microagents 模块文档
- 添加导航面包屑和模块结构说明
- 完善各类微代理的功能描述
- 建立完整的配置和使用指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 17:14:39*