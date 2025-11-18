[根目录](../../CLAUDE.md) > **evaluation**

# Evaluation 评估框架

## 模块职责

evaluation 模块是 OpenHands 平台的核心评估基础设施，提供全面、科学的 AI 代理性能评估框架。该模块支持多种基准测试，包括 SWE-Bench、WebArena、GAIA 等，为 AI 代理的代码生成、问题解决、网页交互等能力提供标准化的评估体系。

## 入口与启动

### 核心评估脚本
- `evaluation/benchmarks/swe_bench/run_infer.py`: SWE-Bench 推理执行器
- `evaluation/benchmarks/swe_bench/eval_infer.py`: SWE-Bench 评估执行器
- `evaluation/utils/shared.py`: 共享评估工具和配置

### 启动方式
```bash
# 运行 SWE-Bench 评估（推荐方式）
./evaluation/benchmarks/swe_bench/scripts/run_infer.sh [model_config] [git-version] [agent] [eval_limit] [max_iter] [num_workers] [dataset] [dataset_split]

# 示例：运行 10 个 SWE-Bench Verified 实例
./evaluation/benchmarks/swe_bench/scripts/run_infer.sh llm.eval_gpt4o HEAD CodeActAgent 10 100 1 princeton-nlp/SWE-bench_Verified test

# 运行多模态评估
./evaluation/benchmarks/swe_bench/scripts/run_infer.sh llm.eval_gpt4_vision HEAD CodeActAgent 10 100 1 princeton-nlp/SWE-bench_Multimodal test

# 评估生成的补丁
./evaluation/benchmarks/swe_bench/scripts/eval_infer.sh [output.jsonl] [instance_id] [dataset_name] [split]
```

## 对外接口

### 支持的评估基准

#### SWE-Bench 系列评估
- **SWE-Bench**: 真实 GitHub Issues 的软件工程修复任务
  - 数据集：`princeton-nlp/SWE-bench`, `princeton-nlp/SWE-bench_Lite`, `princeton-nlp/SWE-bench_Verified`
  - 评估指标：修复成功率、代码质量、测试通过率
  - 支持模式：`swe`（标准模式）、`swt`（测试生成）、`swt-ci`（CI测试）

- **SWE-Gym**: 训练环境扩展版本
  - 交互式学习和训练环境
  - 支持多种难度级别

- **SWE-Bench-Live**: 实时评估环境
  - 持续集成评估
  - 实时代码质量监控

- **SWE-rebench**: 重新评估基准
  - 改进的评估指标和方法
  - 更严格的验证流程

- **Multimodal SWE-Bench**: 多模态评估
  - 支持图像、文本混合输入
  - 视觉理解能力评估

#### 网页交互评估
- **WebArena**: 真实网站任务完成评估
  - 评估指标：任务成功率、操作效率、交互质量
  - 浏览器集成：网页导航、表单填写、信息检索

- **VisualWebArena**: 视觉网页基准
  - 评估指标：视觉理解准确性、任务完成度
  - 多模态集成：图像识别、视觉推理

- **MiniWoB**: 微网页任务基准
  - 评估指标：操作成功率、响应时间
  - 轻量级评估：快速性能测试

#### 综合能力评估
- **GAIA**: 通用 AI 评估框架
  - 多领域任务：跨学科问题解决能力
  - 复杂度分级：不同难度级别的任务

- **ScienceAgentBench**: 科学代理基准
  - 专业性评估：领域知识和研究能力
  - 工具集成：数据分析、可视化工具

#### 其他专业基准
- **Commit0**: 代码提交基准
  - 版本控制：Git 操作和提交质量
  - 代码审查：变更合理性评估

- **EDA**: 探索性数据分析基准
- **Agent Bench**: 代理行为评估
- **Biocoder**: 生物信息学基准
- **Logic Reasoning**: 逻辑推理基准

### 评估配置和参数

#### 迭代评估协议
```bash
# 启用迭代评估（推荐）
export ITERATIVE_EVAL_MODE=true

# 跳过错误继续评估（调试用）
export EVAL_SKIP_MAXIMUM_RETRIES_EXCEEDED=true

# 使用提示文本
export USE_HINT_TEXT=true

# 指定记忆压缩配置
export EVAL_CONDENSER=summarizer_for_eval

# 自定义指令模板
export INSTRUCTION_TEMPLATE_NAME=swe_custom.j2
```

#### 分布式评估
```bash
# 使用 RemoteRuntime 进行云端并行评估
ALLHANDS_API_KEY="YOUR-API-KEY" RUNTIME=remote SANDBOX_REMOTE_RUNTIME_API_URL="https://runtime.eval.all-hands.dev" \
./evaluation/benchmarks/swe_bench/scripts/run_infer.sh llm.eval HEAD CodeActAgent 300 100 16 "princeton-nlp/SWE-bench_Lite" test
```

## 关键依赖与配置

### 评估框架核心
- **streamlit**: Web 界面评估工具
- **evaluate**: Hugging Face 评估库
- **pandas**: 数据处理和分析
- **datasets**: Hugging Face 数据集加载
- **tqdm**: 进度条和可视化

### SWE-Bench 集成
- **swebench**: SWE-Bench 官方评估工具
- **docker**: 容器化评估环境
- **jinja2**: 提示模板引擎
- **toml**: 配置文件解析

### 数据处理和可视化
- **matplotlib/seaborn**: 数据可视化
- **numpy**: 数值计算
- **reportlab**: PDF 报告生成
- **tabulate**: 表格格式化

### 科学计算和工具
- **sympy**: 符号计算
- **joblib**: 并行处理
- **retry**: 重试机制

## 数据模型

### 评估任务定义
```python
class EvaluationTask:
    task_id: str
    instance_id: str
    description: str
    repo: str
    base_commit: str
    problem_statement: str
    hints_text: Optional[str]
    test_patch: str
    version: str
    environment_setup_commit: str
```

### 评估配置
```python
class BenchmarkConfig:
    name: str
    dataset_path: str
    evaluation_metrics: List[str]
    timeout: int
    max_retries: int
    num_workers: int
    dataset_split: str
    mode: str  # 'swe', 'swt', 'swt-ci'
```

### 评估结果
```python
class EvaluationResult:
    instance_id: str
    model_patch: Optional[str]
    model_name_or_path: str
    success: bool
    score: float
    execution_time: float
    cost: float
    token_usage: Dict[str, int]
    error_type: Optional[str]
    detailed_feedback: str
    metrics: Dict[str, float]
```

### SWE-Bench 特定配置
```python
class SWEBenchConfig:
    dataset_type: str  # 'SWE-bench', 'SWE-Gym', 'SWE-bench-Live', 'SWE-rebench', 'Multimodal'
    use_hint_text: bool
    run_with_browsing: bool
    enable_llm_editor: bool
    eval_condenser: str
    instruction_template_name: str
```

## 测试与质量

### 评估流程（SWE-Bench 为例）
1. **任务准备**：
   - 从 HuggingFace 数据集加载实例
   - 准备 Docker 镜像和运行环境
   - 配置代理和模型参数

2. **推理执行**：
   - 启动 OpenHands 运行时环境
   - 执行代理解决软件工程问题
   - 记录详细的执行过程和日志

3. **结果评估**：
   - 应用生成的补丁
   - 运行测试套件验证
   - 评估修复成功率和质量

4. **报告生成**：
   - 生成详细的评估报告
   - 统计分析和可视化
   - 错误分类和根因分析

### 质量保证机制
- **迭代评估协议**：最多 3 次重试，提高结果稳定性
- **容错处理**：自动处理网络错误、API 限流等问题
- **资源管理**：智能的 Docker 镜像管理和清理
- **统计验证**：Bootstrap 方法计算置信区间

### 评估指标体系

#### 主要指标
- **修复成功率**：成功解决的实际问题比例
- **代码质量**：生成代码的可读性和维护性
- **测试通过率**：所有测试用例的通过情况
- **执行效率**：完成任务的时间和资源消耗

#### 详细分析
- **错误分类**：语法错误、逻辑错误、环境问题等
- **成本分析**：API 调用成本、Token 使用量
- **性能分析**：响应时间、并发性能
- **可重现性**：多次运行的一致性

### 科学评估原则
- **标准化流程**：确保所有评估使用一致的流程
- **可重现性**：提供完整的环境和配置复现
- **统计验证**：使用统计学方法验证结果显著性
- **基准对比**：与现有方法和结果进行对比

## 评估工具和脚本

### Docker 管理
```bash
# 拉取所有评估镜像
./evaluation/benchmarks/swe_bench/scripts/docker/pull_all_eval_docker.sh

# 获取 Docker 镜像名称
python3 evaluation/benchmarks/swe_bench/scripts/docker/get_docker_image_names.py
```

### 结果处理
```bash
# 合并最终结果
python3 evaluation/benchmarks/swe_bench/scripts/eval/combine_final_completions.py

# 转换输出格式
python3 evaluation/benchmarks/swe_bench/scripts/eval/convert_oh_output_to_md.py

# 验证成本
python3 evaluation/benchmarks/swe_bench/scripts/eval/verify_costs.py

# 生成摘要报告
python3 evaluation/benchmarks/swe_bench/scripts/eval/summarize_outputs.py
```

### SWT-Bench 集成
```bash
# 转换结果格式
python3 evaluation/benchmarks/swe_bench/scripts/swtbench/convert.py --prediction_file [output.jsonl]

# 运行 SWT-Bench 评估
python3 -m src.main --dataset_name princeton-nlp/SWE-bench_Verified --predictions_path <predictions_path>
```

## 常见问题 (FAQ)

### Q: 如何添加新的评估基准？
A: 创建新的评估目录，实现 `run_infer.py` 和 `eval_infer.py`，遵循现有的接口和模式。

### Q: 如何处理大规模评估？
A: 使用 RemoteRuntime 进行云端并行计算，配置合适的 `num_workers` 和资源因子。

### Q: 如何自定义评估指标？
A: 修改评估脚本中的指标计算逻辑，添加新的度量和分析维度。

### Q: 如何调试评估失败的问题？
A: 检查日志文件，使用 `EVAL_SKIP_MAXIMUM_RETRIES_EXCEEDED=true` 继续评估，分析 `maximum_retries_exceeded.jsonl`。

### Q: 如何确保评估结果的可重现性？
A: 使用固定的随机种子、记录完整的配置信息、使用容器化环境。

## 相关文件清单

### 核心评估框架
- `__init__.py` - 模块初始化
- `README.md` - 模块概述和使用指南

### SWE-Bench 评估系统
- `benchmarks/swe_bench/` - SWE-Bench 完整评估系统
  - `run_infer.py` - 推理执行器（主要评估逻辑）
  - `eval_infer.py` - 评估执行器（补丁验证）
  - `scripts/run_infer.sh` - 推理执行脚本
  - `scripts/eval_infer.sh` - 评估执行脚本
  - `prompts/` - 提示模板目录
  - `loc_eval/` - 代码定位评估
  - `resource/` - 资源配置和常量

### 其他基准测试
- `benchmarks/webarena/` - WebArena 网页交互评估
- `benchmarks/gaia/` - GAIA 综合能力评估
- `benchmarks/miniwob/` - MiniWoB 轻量级评估
- `benchmarks/scienceagentbench/` - 科学代理评估
- `benchmarks/biocoder/` - 生物信息学评估
- `benchmarks/commit0/` - 代码提交评估

### 共享工具和配置
- `utils/shared.py` - 评估工具和配置
- `utils/scripts/` - 实用脚本
- 数据处理和可视化工具

## 变更记录 (Changelog)

### 2025-11-18 18:34:32 - 评估框架深度分析
- **SWE-Bench 系统深度解析**：
  - 完整的评估流程和协议分析
  - 迭代评估机制和容错处理
  - Docker 容器化评估环境管理
  - 多数据集支持（SWE-Bench, SWE-Gym, SWE-Bench-Live, SWE-rebench, Multimodal）

- **评估工具和脚本系统**：
  - 结果处理和格式转换工具
  - SWT-Bench 测试生成评估集成
  - 分布式评估和 RemoteRuntime 支持
  - 成本分析和性能监控工具

- **质量保证和科学评估**：
  - 统计验证和置信区间计算
  - 错误分类和根因分析
  - 可重现性保证机制
  - 基准对比和标准化流程

### 2025-11-18 17:14:39
- 初始化 evaluation 模块文档
- 添加导航面包屑和模块结构说明
- 完善各类评估基准的描述
- 建立完整的评估流程和质量指南
- 添加常见问题和相关文件清单

---

*此文档由 AI 自动生成和维护，最后更新时间：2025-11-18 18:34:32*