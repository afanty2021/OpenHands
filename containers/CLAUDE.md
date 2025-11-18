[根目录](../../CLAUDE.md) > **containers**

# Container Configuration 容器配置

## 模块职责

containers 模块负责 OpenHands 平台的容器化基础设施，提供开发、生产和运行时环境的 Docker 配置。该模块确保平台在不同环境中的一致性部署，支持多架构构建，并提供优化的容器镜像用于评估和生产场景。

## 入口与启动

### 构建脚本
- `build.sh`: 主容器构建脚本，支持多镜像、多平台构建
- `containers/app/entrypoint.sh`: 应用容器入口点
- `containers/app/config.sh`: 应用容器配置脚本

### 启动方式
```bash
# 构建所有镜像
./containers/build.sh --push --load

# 构建特定镜像
./containers/build.sh -i app -o openhands --push

# 开发环境构建
./containers/build.sh -i dev --load
```

## 对外接口

### 容器镜像类型

#### App Container (`containers/app/`)
**用途**: 生产环境应用容器
**基础镜像**: Python 3.12 Slim
**主要特性**:
- 优化的生产环境配置
- 完整的 OpenHands 依赖安装
- 安全增强的运行时环境
- 多架构支持 (x86_64, arm64)

**Dockerfile 关键配置**:
```dockerfile
FROM python:3.12-slim

# 系统依赖安装
RUN apt-get update && apt-get install -y \
    git \
    curl \
    build-essential \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# Python 依赖安装
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 应用代码复制
COPY . .
RUN pip install -e .

# 非 root 用户运行
RUN useradd -m -u 1000 openhands
USER openhands
```

#### Development Container (`containers/dev/`)
**用途**: 开发环境容器
**基础镜像**: Ubuntu 22.04
**主要特性**:
- 完整的开发工具链
- 调试和测试工具
- 热重载支持
- 开发服务器集成

**Docker Compose 配置**:
```yaml
version: '3.8'
services:
  openhands-dev:
    build:
      context: .
      dockerfile: containers/dev/Dockerfile
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
      - "8000:8000"
    environment:
      - NODE_ENV=development
      - PYTHONPATH=/app
```

#### Runtime Container (`containers/runtime/`)
**用途**: 评估和运行时环境
**主要特性**:
- 隔离的执行环境
- 多语言运行时支持
- 安全沙盒配置
- 资源限制和监控

## 关键依赖与配置

### Docker 构建系统
- **Docker BuildKit**: 高级构建功能
- **Buildx**: 多平台构建支持
- **Multi-stage builds**: 优化镜像大小
- **Layer caching**: 构建性能优化

### 容器编排
- **Docker Compose**: 本地开发环境
- **Kubernetes**: 生产环境编排
- **Helm Charts**: K8s 部署模板
- **Docker Swarm**: 简单集群部署

### 安全配置
- **Non-root 用户**: 安全运行用户
- **最小权限原则**: 最小化容器权限
- **安全扫描**: 镜像漏洞扫描
- **签名验证**: 镜像完整性验证

## 构建和部署

### 多平台构建
```bash
# 构建并推送到多平台
./containers/build.sh -i app -o openhands --push \
  --platform linux/amd64,linux/arm64

# 本地多架构构建
docker buildx build --platform linux/amd64,linux/arm64 \
  -t openhands/app:latest .
```

### CI/CD 集成
- **GitHub Actions**: 自动化构建和发布
- **自动化测试**: 容器化测试环境
- **镜像扫描**: 安全漏洞检测
- **自动部署**: 生产环境滚动更新

### 镜像优化策略
- **Alpine 基础镜像**: 最小化镜像大小
- **多阶段构建**: 分离构建和运行时环境
- **.dockerignore**: 排除不必要文件
- **层缓存优化**: 优化 Dockerfile 层结构

## 配置管理

### 环境变量配置
```bash
# 应用配置
OH_API_BASE_URL=https://api.openhands.ai
OH_DEFAULT_LLM=gpt-4o

# 容器配置
CONTAINER_USER=openhands
CONTAINER_UID=1000
CONTAINER_GID=1000

# 构建配置
BUILDPLATFORM=linux/amd64
TARGETPLATFORM=linux/amd64
```

### 运行时配置
```bash
# 资源限制
MEMORY_LIMIT=4g
CPU_LIMIT=2
DISK_SPACE=10g

# 网络配置
NETWORK_MODE=bridge
PORT_MAPPING=3000:3000
EXPOSED_PORTS=3000,8000,8080
```

## 安全和合规

### 容器安全
- **镜像扫描**: Trivy/Clair 集成
- **运行时保护**: Falco/Sysdig 安全监控
- **网络隔离**: 容器网络分段
- **文件系统保护**: 只读根文件系统

### 合规性检查
- **CIS Docker Benchmark**: Docker 安全基线
- **NIST 合规**: 安全配置标准
- **SOC 2 Type II**: 安全控制验证
- **GDPR 合规**: 数据保护规定

## 监控和运维

### 容器监控
- **Prometheus**: 指标收集
- **Grafana**: 可视化监控仪表板
- **cAdvisor**: 容器资源监控
- **ELK Stack**: 日志聚合和分析

### 健康检查
```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

### 日志管理
- **结构化日志**: JSON 格式日志输出
- **日志聚合**: 集中式日志收集
- **日志轮转**: 自动日志清理
- **审计日志**: 操作审计跟踪

## 性能优化

### 镜像优化
- **镜像大小**: 最小化镜像体积
- **启动时间**: 优化容器启动速度
- **内存使用**: 减少运行时内存占用
- **网络性能**: 优化容器网络配置

### 构建优化
- **并行构建**: 多阶段并行构建
- **缓存策略**: 智能层缓存
- **依赖管理**: 优化依赖安装顺序
- **构建加速**: BuildKit 特性利用

## 常见问题 (FAQ)

### Q: 如何处理容器的持久化数据？
A: 使用 Docker volumes 或 bind mounts，配置适当的备份策略，确保数据持久化和一致性。

### Q: 如何优化容器的启动时间？
A: 使用多阶段构建，优化依赖安装，利用镜像层缓存，减少启动时的初始化操作。

### Q: 如何实现容器的自动扩缩容？
A: 使用 Kubernetes HPA (Horizontal Pod Autoscaler)，配置 CPU、内存或自定义指标的自动扩缩容策略。

### Q: 如何处理容器安全漏洞？
A: 定期更新基础镜像，使用自动化安全扫描，及时修复已知漏洞，实施最小权限原则。

### Q: 如何优化多平台构建的性能？
A: 使用 BuildKit 缓存挂载，启用并行构建，优化 Dockerfile 层结构，使用适当的构建器实例。

## 相关文件清单

### 构建脚本
- `build.sh` - 主容器构建脚本
- `containers/app/entrypoint.sh` - 应用容器入口点
- `containers/app/config.sh` - 应用容器配置

### Docker 配置
- `containers/app/Dockerfile` - 生产环境容器定义
- `containers/dev/Dockerfile` - 开发环境容器定义
- `containers/dev/compose.yml` - Docker Compose 配置
- `containers/dev/README.md` - 开发环境说明
- `containers/runtime/README.md` - 运行时环境说明

### CI/CD 集成
- `.github/workflows/ghcr-build.yml` - GitHub Actions 构建流程
- `.github/workflows/py-tests.yml` - 容器化测试
- `.github/workflows/ui-build.yml` - UI 构建

### 配置文件
- `containers/app/config.sh` - 应用配置脚本
- `containers/dev/dev.sh` - 开发环境脚本
- `containers/runtime/config.sh` - 运行时配置

## 变更记录 (Changelog)

### 2025-11-18 19:11:18 - 容器配置模块初始化
- **容器化基础设施建立**：
  - 完整的 Docker 构建系统分析
  - 多平台构建和多架构支持
  - 开发、生产、运行时环境配置
  - 安全和性能优化策略

- **CI/CD 集成分析**：
  - GitHub Actions 自动化构建流程
  - 容器化测试和部署策略
  - 镜像扫描和安全检查
  - 多平台发布和分发机制

- **运维和监控体系**：
  - Prometheus + Grafana 监控栈
  - 健康检查和自动恢复
  - 日志管理和审计跟踪
  - 容器安全和合规性框架

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 19:11:18*