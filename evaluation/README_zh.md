本文档介绍如何使用 [SWE-bench](https://www.swebench.com/)、[SWE-bench-Live](https://swe-bench-live.github.io/) 和 [Multi-SWE-bench](https://multi-swe-bench.github.io/) 评估 [Trae Agent](https://github.com/bytedance/trae-agent)。

## 概述

**SWE-bench** 是一个基准测试，用于评估语言模型在实际软件工程任务中的表现。它包含来自流行 Python 仓库的 GitHub issue，这些 issue 已由人类开发者解决。该基准测试评估智能体是否能够生成正确的补丁来修复问题。

**SWE-bench-Live** 是一个实时的 issue 解决基准测试，旨在评估 AI 系统完成实际软件工程任务的能力。得益于我们的自动化数据集构建管道，我们计划每月更新 SWE-bench-Live，为社区提供最新的任务实例，并支持严格且无污染的评估。

**Multi-SWE-bench** 是一个多语言的 issue 解决基准测试。它涵盖 7 种语言（Java、TypeScript、JavaScript、Go、Rust、C 和 C++），包含由 68 位专家注释者从 2,456 个候选实例中精心挑选的 1,632 个高质量实例，以确保可靠性。

评估过程包括：
1. **设置**：使用 Docker 容器准备评估环境
2. **执行**：在实例上运行 Trae Agent 生成补丁
3. **评估**：使用测试框架将生成的补丁与标准答案进行对比测试

## 先决条件

在运行评估之前，请确保您具备以下条件：

- **Docker**：用于容器化评估环境
- **Python 3.12+**：用于运行评估脚本
- **Git**：用于克隆仓库
- **足够的磁盘空间**：每个实例的 Docker 镜像可能有数 GB 大小
- **API 密钥**：Trae Agent 使用的 OpenAI/Anthropic API 密钥

## 设置说明

确保安装评估所需的额外依赖，并在 `evaluation` 目录中运行脚本：

```bash
uv sync --extra evaluation
cd evaluation
```

### 1. 克隆和设置基准测试框架

`setup.sh` 脚本自动化了基准测试框架的设置：

```bash
chmod +x setup.sh
./setup.sh [swe_bench|swe_bench_live|multi_swe_bench]
```

- `swe_bench`：设置 SWE-Bench
- `swe_bench_live`：设置 SWE-Bench-Live
- `multi_swe_bench`：设置 Multi-SWE-Bench

此脚本会：
- 克隆基准测试仓库
- 检出特定的提交以确保可重现性（这是撰写本文档时最新的提交哈希值）
- 创建 Python 虚拟环境
- 安装基准测试框架

### 2. 配置 Trae Agent

确保您的 `trae_config.yaml` 文件已正确配置有效的 API 密钥：

```yaml
agents:
  trae_agent:
    enable_lakeview: false
    model: trae_agent_model  # Trae Agent 的模型配置名称
    max_steps: 200  # 智能体最大步数
    tools:  # Trae Agent 使用的工具
      - bash
      - str_replace_based_edit_tool
      - sequentialthinking
      - task_done

model_providers:  # 模型提供商配置
  anthropic:
    api_key: your_anthropic_api_key
    provider: anthropic
  openai:
    api_key: your_openai_api_key
    provider: openai

models:
  trae_agent_model:
    model_provider: anthropic
    model: claude-sonnet-4-20250514
    max_tokens: 4096
    temperature: 0.5
    top_p: 0.9
    top_k: 40
    max_retries: 1
    parallel_tool_calls: 1
```

### 3. 可选：Docker 环境配置

如果您需要自定义环境变量，请创建 `docker_env_config.json` 文件：

```json
{
  "preparation_env": {
    "HTTP_PROXY": "http://proxy.example.com:8080",
    "HTTPS_PROXY": "https://proxy.example.com:8080"
  },
  "experiment_env": {
    "CUSTOM_VAR": "value"
  }
}
```

## 使用方法

### 基本用法

评估脚本 `run_evaluation.py` 提供多种操作模式：

```bash
# 在 SWE-bench_Verified 的所有实例上运行评估
python run_evaluation.py --dataset SWE-bench_Verified --working-dir ./trae-workspace

# 在特定实例上运行评估
python run_evaluation.py --instance_ids django__django-12345 scikit-learn__scikit-learn-67890

# 使用自定义配置运行
python run_evaluation.py --config-file trae_config.yaml --run-id experiment-1
```

### 可用的基准测试和数据集

**SWE-bench**
- **SWE-bench_Verified**
- **SWE-bench_Lite**
- **SWE-bench**

**SWE-bench-Live**：
- **SWE-bench-Live/lite**
- **SWE-bench-Live/verified**
- **SWE-bench-Live/full**

**Multi-SWE-bench**：
- **Multi-SWE-bench-flash**（请从 https://huggingface.co/datasets/ByteDance-Seed/Multi-SWE-bench-flash/tree/main 下载 `multi_swe_bench_flash.jsonl` 并将其放在 `evaluation` 目录中。）
- **Multi-SWE-bench_mini**（请从 https://huggingface.co/datasets/ByteDance-Seed/Multi-SWE-bench_mini/tree/main 下载 `multi_swe_bench_mini.jsonl` 并将其放在 `evaluation` 目录中。）

### 评估模式

脚本支持三种模式：

1. **`expr`**（仅生成）：生成补丁但不进行评估
2. **`eval`**（仅评估）：评估已有的补丁
3. **`e2e`**（端到端）：同时生成和评估补丁（默认）

```bash
# 仅生成补丁
python run_evaluation.py --mode expr --dataset SWE-bench_Verified

# 仅评估已有的补丁
python run_evaluation.py --mode eval --benchmark-harness-path ./SWE-bench

# 端到端评估（默认）
python swebench.py --mode e2e --benchmark-harness-path ./SWE-bench
```

### 完整命令参考

```bash
python run_evaluation.py \
  --benchmark SWE-bench \
  --dataset SWE-bench_Verified \
  --config-file ./trae_config.yaml \
  --run-id experiment-1 \
  --benchmark-harness-path ./SWE-bench \
  --docker-env-config ./docker_env_config.json \
  --mode e2e \
  --max_workers 4 \
  --instance_ids astropy__astropy-13453
```

**参数说明：**
- `--benchmark`：使用的基准测试
- `--dataset`：使用的数据集
- `--config-file`：Trae Agent 配置文件
- `--run-id`：基准测试评估的运行 ID
- `--benchmark-harness-path`：SWE-bench 框架的路径（评估必需）
- `--docker-env-config`：Docker 环境配置文件
- `--mode`：评估模式（`e2e`、`expr`、`eval`）
- `--max_workers`：用于并行执行的最大工作进程数
- `--instance_ids`：要使用的实例 ID

## 工作原理

### 1. 镜像准备

脚本首先检查所需的 Docker 镜像：
- 每个实例都有特定的 Docker 镜像
- 如果本地不存在，会自动拉取镜像
- 使用基础 Ubuntu 镜像准备 Trae Agent

### 2. Trae Agent 准备

脚本在 Docker 容器中构建 Trae Agent：
- 创建构件（`trae-agent.tar`、`uv.tar`、`uv_shared.tar`）
- 这些构件在所有实例间复用，以提高效率

### 3. 实例执行

对于每个实例：
1. **容器设置**：准备带有实例环境的 Docker 容器
2. **问题描述**：将 GitHub issue 描述写入文件
3. **Trae Agent 执行**：运行 Trae Agent 生成补丁
4. **补丁收集**：保存生成的补丁以供评估

### 4. 评估

使用基准测试框架：
1. **补丁收集**：将所有生成的补丁收集到 `predictions.json`
2. **测试执行**：在 Docker 容器中针对测试套件运行补丁
3. **结果生成**：生成带有通过/失败状态的评估结果

## 理解结果

### 输出文件

评估会在工作目录中创建多个文件：

```
results/{benchmark}_{dataset}_{run_id}/
├── predictions.json              # 用于评估的生成补丁
├── results.json                  # 最终评估结果
├── {instance_id}/                # 每个实例的文件夹
│   ├── problem_statement.txt     # GitHub issue 描述
│   ├── {instance_id}.patch       # 生成的补丁
│   ├── {instance_id}.json        # 轨迹文件
│   └── ...
trae-workspace/
├── trae_config.yaml              # Trae Agent 配置文件
├── trae-agent.tar                # Trae Agent 构建构件
├── uv.tar                        # UV 二进制文件
└── uv_shared.tar                 # UV 共享文件
```