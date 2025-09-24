[![arXiv:2507.23370](https://img.shields.io/badge/技术报告-arXiv%3A2507.23370-b31a1b)](https://arxiv.org/abs/2507.23370) [![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)](https://www.python.org/downloads/) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Pre-commit](https://github.com/bytedance/trae-agent/actions/workflows/pre-commit.yml/badge.svg)](https://github.com/bytedance/trae-agent/actions/workflows/pre-commit.yml)
[![Unit Tests](https://github.com/bytedance/trae-agent/actions/workflows/unit-test.yml/badge.svg)](https://github.com/bytedance/trae-agent/actions/workflows/unit-test.yml)
[![Discord](https://img.shields.io/discord/1320998163615846420?label=加入%20Discord&color=7289DA)](https://discord.gg/VwaQ4ZBHvC)

**Trae Agent** 是一个基于 LLM 的通用软件工程任务代理。它提供了强大的 CLI 界面，能够理解自然语言指令并使用各种工具和 LLM 提供商执行复杂的软件工程工作流。

有关技术细节，请参阅 [我们的技术报告](https://arxiv.org/abs/2507.23370)。

**项目状态：** 项目仍在积极开发中。如果您希望帮助我们改进 Trae Agent，请参阅 [docs/roadmap.md](docs/roadmap.md) 和 [CONTRIBUTING](CONTRIBUTING.md)。

**与其他 CLI 代理的区别：** Trae Agent 提供了透明、模块化的架构，研究人员和开发人员可以轻松修改、扩展和分析，使其成为 **研究 AI 代理架构、进行消融研究和开发新型代理能力** 的理想平台。这种 **_研究友好的设计_** 使学术界和开源社区能够为基础代理框架做出贡献并在此基础上构建，从而在快速发展的 AI 代理领域促进创新。

## ✨ 特性

- 🌊 **Lakeview**：为代理步骤提供简短而简洁的摘要
- 🤖 **多 LLM 支持**：支持 OpenAI、Anthropic、Doubao、Azure、OpenRouter、Ollama 和 Google Gemini API
- 🛠️ **丰富的工具生态系统**：文件编辑、bash 执行、顺序思考等
- 🎯 **交互模式**：迭代开发的对话界面
- 📊 **轨迹记录**：所有代理操作的详细日志，用于调试和分析
- ⚙️ **灵活配置**：基于 YAML 的配置，支持环境变量
- 🚀 **易于安装**：简单的基于 pip 的安装

## 🚀 安装

### 要求
- UV (https://docs.astral.sh/uv/)
- 您选择的提供商的 API 密钥（OpenAI、Anthropic、Google Gemini、OpenRouter 等）

### 设置

```bash
git clone https://github.com/bytedance/trae-agent.git
cd trae-agent
uv sync --all-extras
source .venv/bin/activate
```

## ⚙️ 配置

### YAML 配置（推荐）

1. 复制示例配置文件：
   ```bash
   cp trae_config.yaml.example trae_config.yaml
   ```

2. 编辑 `trae_config.yaml` 配置您的 API 凭据和偏好设置：

```yaml
agents:
  trae_agent:
    enable_lakeview: true
    model: trae_agent_model  # Trae Agent 的模型配置名称
    max_steps: 200  # 代理步骤的最大数量
    tools:  # 与 Trae Agent 一起使用的工具
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
```

**注意：** `trae_config.yaml` 文件被 git 忽略，以保护您的 API 密钥。

### 使用基础 URL
在某些情况下，我们需要为 API 使用自定义 URL。只需在 `provider` 后添加 `base_url` 字段，以下面的配置为例：

```
openai:
    api_key: your_openrouter_api_key
    provider: openai
    base_url: https://openrouter.ai/api/v1
```
**注意：** 对于字段格式，仅使用空格。不允许使用制表符（\t）。

### 环境变量（替代方案）

您也可以使用环境变量配置 API 密钥并将其存储在 .env 文件中：

```bash
export OPENAI_API_KEY="your-openai-api-key"
export OPENAI_BASE_URL="your-openai-base-url"
export ANTHROPIC_API_KEY="your-anthropic-api-key"
export ANTHROPIC_BASE_URL="your-anthropic-base-url"
export GOOGLE_API_KEY="your-google-api-key"
export GOOGLE_BASE_URL="your-google-base-url"
export OPENROUTER_API_KEY="your-openrouter-api-key"
export OPENROUTER_BASE_URL="https://openrouter.ai/api/v1"
export DOUBAO_API_KEY="your-doubao-api-key"
export DOUBAO_BASE_URL="https://ark.cn-beijing.volces.com/api/v3/"
```

### MCP 服务（可选）

要启用模型上下文协议（MCP）服务，请在配置中添加 `mcp_servers` 部分：

```yaml
mcp_servers:
  playwright:
    command: npx
    args:
      - "@playwright/mcp@0.0.27"
```

**配置优先级：** 命令行参数 > 配置文件 > 环境变量 > 默认值

**旧版 JSON 配置：** 如果使用较旧的 JSON 格式，请参阅 [docs/legacy_config.md](docs/legacy_config.md)。我们建议迁移到 YAML。

## 📖 使用

### 基本命令

```bash
# 简单任务执行
trae-cli run "创建一个 hello world Python 脚本"

# 检查配置
trae-cli show-config

# 交互模式
trae-cli interactive
```

### 特定提供商示例

```bash
# OpenAI
trae-cli run "修复 main.py 中的错误" --provider openai --model gpt-4o

# Anthropic
trae-cli run "添加单元测试" --provider anthropic --model claude-sonnet-4-20250514

# Google Gemini
trae-cli run "优化这个算法" --provider google --model gemini-2.5-flash

# OpenRouter（访问多个提供商）
trae-cli run "审核这段代码" --provider openrouter --model "anthropic/claude-3-5-sonnet"
trae-cli run "生成文档" --provider openrouter --model "openai/gpt-4o"

# Doubao
trae-cli run "重构数据库模块" --provider doubao --model doubao-seed-1.6

# Ollama（本地模型）
trae-cli run "为这段代码添加注释" --provider ollama --model qwen3
```

### 高级选项

```bash
# 自定义工作目录
trae-cli run "为 utils 模块添加测试" --working-dir /path/to/project

# 保存执行轨迹
trae-cli run "调试身份验证" --trajectory-file debug_session.json

# 强制生成补丁
trae-cli run "更新 API 端点" --must-patch

# 自定义设置的交互模式
trae-cli interactive --provider openai --model gpt-4o --max-steps 30
```

## Docker 模式命令
### 准备
**重要**：您需要确保 Docker 在您的环境中已配置。

### 使用
```bash
# 指定 Docker 镜像在新容器中运行任务
trae-cli run "为 utils 模块添加测试" --docker-image python:3.11

# 指定 Docker 镜像在新容器中运行任务并挂载目录
trae-cli run "编写一个打印 helloworld 的脚本" --docker-image python:3.12 --working-dir test_workdir/

# 通过 ID 附加到现有 Docker 容器（`--working-dir` 与 `--docker-container-id` 不兼容）
trae-cli run "更新 API 端点" --docker-container-id 91998a56056c

# 指定 Dockerfile 的绝对路径来构建环境
trae-cli run "调试身份验证" --dockerfile-path test_workspace/Dockerfile

# 指定本地 Docker 镜像文件（tar 存档）的路径来加载
trae-cli run "修复 main.py 中的错误" --docker-image-file test_workspace/trae_agent_custom.tar

# 完成任务后删除 Docker 容器（保持默认）
trae-cli run "为 utils 模块添加测试" --docker-image python:3.11 --docker-keep false
```

### 交互模式命令

在交互模式下，您可以使用：
- 输入任何任务描述来执行它
- `status` - 显示代理信息
- `help` - 显示可用命令
- `clear` - 清除屏幕
- `exit` 或 `quit` - 结束会话

## 🛠️ 高级功能

### 可用工具

Trae Agent 为软件工程任务提供了全面的工具包，包括文件编辑、bash 执行、结构化思考和任务完成。有关所有可用工具及其功能的详细信息，请参阅 [docs/tools.md](docs/tools.md)。

### 轨迹记录

Trae Agent 自动记录详细的执行轨迹，用于调试和分析：

```bash
# 自动生成的轨迹文件
trae-cli run "调试身份验证模块"
# 保存到：trajectories/trajectory_YYYYMMDD_HHMMSS.json

# 自定义轨迹文件
trae-cli run "优化数据库查询" --trajectory-file optimization_debug.json
```

轨迹文件包含 LLM 交互、代理步骤、工具使用和执行元数据。有关更多详细信息，请参阅 [docs/TRAJECTORY_RECORDING.md](docs/TRAJECTORY_RECORDING.md)。

## 🔧 开发

### 贡献

有关贡献指南，请参阅 [CONTRIBUTING.md](CONTRIBUTING.md)。

### 故障排除

**导入错误：**
```bash
PYTHONPATH=. trae-cli run "您的任务"
```

**API 密钥问题：**
```bash
# 验证 API 密钥
echo $OPENAI_API_KEY
trae-cli show-config
```

**命令未找到：**
```bash
uv run trae-cli run "您的任务"
```

**权限错误：**
```bash
chmod +x /path/to/your/project
```

## 📄 许可证

本项目根据 MIT 许可证授权 - 有关详细信息，请参阅 [LICENSE](LICENSE) 文件。

## ✍️ 引用

```bibtex
@article{traeresearchteam2025traeagent,
      title={Trae Agent: An LLM-based Agent for Software Engineering with Test-time Scaling},
      author={Trae Research Team and Pengfei Gao and Zhao Tian and Xiangxin Meng and Xinchen Wang and Ruida Hu and Yuanan Xiao and Yizhou Liu and Zhao Zhang and Junjie Chen and Cuiyun Gao and Yun Lin and Yingfei Xiong and Chao Peng and Xia Liu},
      year={2025},
      eprint={2507.23370},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2507.23370},
}
```

## 🙏 致谢

我们感谢 Anthropic 构建的 [anthropic-quickstart](https://github.com/anthropics/anthropic-quickstarts) 项目，该项目为工具生态系统提供了宝贵的参考。