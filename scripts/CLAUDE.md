[根目录](../../CLAUDE.md) > **scripts**

# Utility Scripts 实用脚本

## 模块职责

scripts 模块包含 OpenHands 平台的开发、部署和维护脚本，提供自动化的工具集来简化开发流程、配置管理、API 文档生成等常用任务。这些脚本是平台工程化的重要组成部分，确保开发效率和操作一致性。

## 入口与启动

### 核心脚本
- `dump_config_schema.py`: 配置模式导出工具
- `update_openapi.py`: OpenAPI 文档自动生成工具

### 启动方式
```bash
# 导出配置模式
python scripts/dump_config_schema.py

# 生成 API 文档
python scripts/update_openapi.py

# 获取帮助信息
python scripts/dump_config_schema.py --help
python scripts/update_openapi.py --help
```

## 对外接口

### 配置管理工具

#### dump_config_schema.py
**用途**: 导出 OpenHands 配置模式的 JSON Schema 文档
**功能特性**:
- 自动生成配置文件结构文档
- 支持 Pydantic 模型解析
- 提供配置项类型和约束信息
- 生成配置验证规则

**使用示例**:
```bash
# 导出完整配置模式
python scripts/dump_config_schema.py --output config_schema.json

# 导出特定模块配置
python scripts/dump_config_schema.py --module LLMConfig --output llm_config.json

# 包含默认值和示例
python scripts/dump_config_schema.py --include-examples --output config_with_examples.json
```

**输出格式**:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "OpenHands Configuration Schema",
  "type": "object",
  "properties": {
    "llm": {
      "$ref": "#/definitions/LLMConfig"
    },
    "security": {
      "$ref": "#/definitions/SecurityConfig"
    }
  },
  "definitions": {
    "LLMConfig": {
      "type": "object",
      "properties": {
        "model": {"type": "string"},
        "api_key": {"type": "string"},
        "max_tokens": {"type": "integer", "default": 4096}
      }
    }
  }
}
```

### API 文档工具

#### update_openapi.py
**用途**: 自动生成和更新 OpenAPI 规范文档
**功能特性**:
- 解析 FastAPI 路由和模型
- 生成标准 OpenAPI 3.0 规范
- 支持 API 版本管理
- 自动更新前端 API 客户端

**使用示例**:
```bash
# 生成完整 API 文档
python scripts/update_openapi.py --output api/openapi.json

# 更新特定版本 API
python scripts/update_openapi.py --version v1 --output api/v1/openapi.json

# 生成 TypeScript 客户端
python scripts/update_openapi.py --generate-ts-client --output frontend/src/api/generated
```

**生成的 OpenAPI 示例**:
```yaml
openapi: 3.0.0
info:
  title: OpenHands API
  version: 1.0.0
  description: AI Agent Software Development Platform API
paths:
  /api/conversations:
    post:
      summary: Create new conversation
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ConversationRequest'
      responses:
        200:
          description: Conversation created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ConversationResponse'
```

## 关键依赖与配置

### 配置解析
- **Pydantic**: 数据验证和设置管理
- **jsonschema**: JSON Schema 验证
- **toml**: TOML 配置文件解析
- **yaml**: YAML 配置文件支持

### API 文档生成
- **fastapi**: Web 框架和自动文档
- **pydantic**: 数据模型和验证
- **jsonref**: JSON 引用解析
- **inflection**: 字符串格式化工具

### 生成器工具
- **Jinja2**: 模板引擎
- **datamodel-code-generator**: 数据模型代码生成
- **openapi-generator**: API 客户端代码生成

## 开发自动化

### 构建前脚本
```bash
# 生成配置文档
python scripts/dump_config_schema.py --output docs/config_schema.json

# 更新 API 文档
python scripts/update_openapi.py --output api/openapi.json

# 验证配置文件
python scripts/validate_config.py --config config/default.toml
```

### 开发服务器启动
```bash
#!/bin/bash
# 开发环境启动脚本

# 生成最新配置
python scripts/dump_config_schema.py

# 更新 API 文档
python scripts/update_openapi.py

# 启动开发服务器
cd frontend && npm run dev &
cd openhands && python -m openhands.server.listen_socket
```

### 测试数据生成
```bash
# 生成测试配置
python scripts/generate_test_config.py --output tests/fixtures/test_config.json

# 生成模拟 API 响应
python scripts/generate_mock_data.py --count 100 --output tests/mock_data.json
```

## 部署自动化

### 生产环境准备
```bash
#!/bin/bash
# 生产部署准备脚本

# 验证配置
python scripts/dump_config_schema.py --validate-config config/production.toml

# 生成 API 文档
python scripts/update_openapi.py --version v1 --output dist/api/v1/openapi.json

# 压缩静态资源
python scripts/compress_assets.py --input frontend/build --output dist/
```

### 容器构建辅助
```bash
#!/bin/bash
# Docker 构建前准备

# 生成运行时配置
python scripts/generate_runtime_config.py --env production --output .env

# 验证依赖
python scripts/verify_dependencies.py --requirements requirements.txt

# 构建容器
./containers/build.sh -i app --push
```

## 配置管理系统

### 配置模板生成
```python
# scripts/generate_config_template.py
import json
from pathlib import Path

def generate_config_template():
    """生成配置模板文件"""
    schema = load_config_schema()

    template = {
        "llm": {
            "model": schema["definitions"]["LLMConfig"]["properties"]["model"]["default"],
            "max_tokens": schema["definitions"]["LLMConfig"]["properties"]["max_tokens"]["default"]
        },
        "security": {
            "enabled": True,
            "level": "standard"
        }
    }

    with open("config/template.toml", "w") as f:
        toml.dump(template, f)
```

### 配置验证工具
```python
# scripts/validate_config.py
import jsonschema
import toml
from pathlib import Path

def validate_config(config_path: str, schema_path: str = "config_schema.json"):
    """验证配置文件是否符合 schema"""
    with open(schema_path) as f:
        schema = json.load(f)

    with open(config_path) as f:
        config = toml.load(f)

    jsonschema.validate(config, schema)
    print(f"✅ {config_path} 验证通过")
```

## API 文档管理

### 版本控制
```bash
# 生成多版本 API 文档
python scripts/update_openapi.py \
  --version v1 \
  --output api/v1/openapi.json \
  --changelog CHANGELOG_v1.md

python scripts/update_openapi.py \
  --version v2 \
  --output api/v2/openapi.json \
  --changelog CHANGELOG_v2.md
```

### 客户端代码生成
```bash
# 生成 TypeScript 客户端
python scripts/update_openapi.py \
  --generate-ts-client \
  --output frontend/src/api/generated \
  --client-name openhands-api

# 生成 Python 客户端
python scripts/update_openapi.py \
  --generate-python-client \
  --output python-client \
  --package-name openhands-client
```

### 文档站点生成
```bash
# 生成 Swagger UI 静态站点
python scripts/generate_docs_site.py \
  --openapi api/openapi.json \
  --output docs/api \
  --theme material
```

## 监控和维护

### 配置监控
```bash
# 监控配置变更
python scripts/monitor_config_changes.py \
  --config-dir config/ \
  --notify-email admin@openhands.ai
```

### API 健康检查
```bash
# API 端点健康检查
python scripts/api_health_check.py \
  --base-url https://api.openhands.ai \
  --endpoints /health,/api/conversations,/api/agents \
  --timeout 30
```

### 性能基准测试
```bash
# API 性能测试
python scripts/api_benchmark.py \
  --url https://api.openhands.ai \
  --concurrent 10 \
  --requests 1000 \
  --output benchmark_results.json
```

## 集成和扩展

### CI/CD 集成
```yaml
# .github/workflows/scripts-validation.yml
name: Scripts Validation
on: [push, pull_request]

jobs:
  validate-scripts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.12'

      - name: Validate configuration schema
        run: |
          python scripts/dump_config_schema.py --validate-configs

      - name: Generate API documentation
        run: |
          python scripts/update_openapi.py --dry-run
```

### IDE 集成
```bash
# VS Code 任务配置 (.vscode/tasks.json)
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Generate Config Schema",
      "type": "shell",
      "command": "python scripts/dump_config_schema.py",
      "group": "build"
    },
    {
      "label": "Update API Docs",
      "type": "shell",
      "command": "python scripts/update_openapi.py",
      "group": "build"
    }
  ]
}
```

## 常见问题 (FAQ)

### Q: 如何添加新的配置管理脚本？
A: 遵循现有的脚本模式，添加适当的错误处理和日志记录，更新相关的文档和测试用例。

### Q: 如何自定义 API 文档生成？
A: 修改 `update_openapi.py` 脚本，添加自定义的生成器和模板，扩展 OpenAPI 规范的附加信息。

### Q: 如何处理配置文件的加密？
A: 扩展配置管理脚本，集成加密库（如 cryptography），实现敏感配置的加密存储和解密加载。

### Q: 如何实现多环境配置管理？
A: 创建环境特定的配置文件和模板，在脚本中添加环境检测和配置合并逻辑。

### Q: 如何集成第三方工具和插件？
A: 在脚本中添加插件接口，支持动态加载和执行第三方工具，提供扩展点机制。

## 相关文件清单

### 配置管理
- `dump_config_schema.py` - 配置模式导出工具
- `generate_config_template.py` - 配置模板生成器
- `validate_config.py` - 配置验证工具
- `monitor_config_changes.py` - 配置变更监控

### API 文档
- `update_openapi.py` - OpenAPI 文档生成工具
- `generate_docs_site.py` - 文档站点生成器
- `api_health_check.py` - API 健康检查工具
- `api_benchmark.py` - API 性能测试工具

### 部署和维护
- `generate_runtime_config.py` - 运行时配置生成器
- `verify_dependencies.py` - 依赖验证工具
- `compress_assets.py` - 静态资源压缩工具
- `generate_test_data.py` - 测试数据生成器

## 变更记录 (Changelog)

### 2025-11-18 19:11:18 - 实用脚本模块初始化
- **脚本系统建立**：
  - 配置模式导出和验证工具
  - OpenAPI 文档自动生成系统
  - 开发和部署自动化脚本
  - 监控和维护工具集

- **工程化工具集成**：
  - CI/CD 流程集成脚本
  - IDE 开发环境配置
  - 多环境配置管理
  - 性能监控和基准测试

- **扩展性设计**：
  - 插件接口和扩展点
  - 第三方工具集成框架
  - 自定义生成器和模板系统
  - 配置加密和安全管理

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 19:11:18*