# 轨迹记录功能

本文档描述了Trae Agent项目中添加的轨迹记录功能。该系统捕获有关LLM交互和代理执行步骤的详细信息，用于分析、调试和审计目的。

## 概述

轨迹记录系统捕获：

- **原始LLM交互**：各种提供商（包括Anthropic、OpenAI、Google Gemini、Azure等）的输入消息、响应、令牌使用情况和工具调用。
- **代理执行步骤**：状态转换、工具调用、工具结果、反思和错误
- **元数据**：任务描述、时间戳、模型配置和执行指标

## 关键组件

### 1. TrajectoryRecorder (`trae_agent/utils/trajectory_recorder.py`)

处理将轨迹数据记录到JSON文件的核心类。

**主要方法：**

- `start_recording()`：使用任务元数据初始化记录
- `record_llm_interaction()`：捕获LLM请求/响应对
- `record_agent_step()`：捕获代理执行步骤
- `finalize_recording()`：完成记录并保存最终结果

### 2. 客户端集成

所有支持的LLM客户端在附加轨迹记录器时会自动记录交互。

**Anthropic客户端** (`trae_agent/utils/anthropic_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="anthropic",
        model=model_parameters.model,
        tools=tools
    )
```

**OpenAI客户端** (`trae_agent/utils/openai_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="openai",
        model=model_parameters.model,
        tools=tools
    )
```

**Google Gemini客户端** (`trae_agent/utils/google_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="google",
        model=model_parameters.model,
        tools=tools,
    )
```

**Azure客户端** (`trae_agent/utils/azure_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="azure",
        model=model_parameters.model,
        tools=tools,
    )
```

**Doubao客户端** (`trae_agent/utils/doubao_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="doubao",
        model=model_parameters.model,
        tools=tools,
    )
```

**Ollama客户端** (`trae_agent/utils/ollama_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="openai", # Ollama客户端为了一致性使用OpenAI的提供商名称
        model=model_parameters.model,
        tools=tools,
    )
```

**OpenRouter客户端** (`trae_agent/utils/openrouter_client.py`)：

```python
# 如果记录器可用，记录轨迹
if self.trajectory_recorder:
    self.trajectory_recorder.record_llm_interaction(
        messages=messages,
        response=llm_response,
        provider="openrouter",
        model=model_parameters.model,
        tools=tools,
    )
```

### 3. 代理集成

基础Agent类自动记录执行步骤：

```python
# 记录代理步骤
if self.trajectory_recorder:
    self.trajectory_recorder.record_agent_step(
        step_number=step.step_number,
        state=step.state.value,
        llm_messages=messages,
        llm_response=step.llm_response,
        tool_calls=step.tool_calls,
        tool_results=step.tool_results,
        reflection=step.reflection,
        error=step.error
    )
```

## 使用方法

### 命令行使用

#### 基本记录（自动生成文件名）

```bash
trae run "创建一个hello world Python脚本"
# 轨迹保存至：trajectories/trajectory_20250612_220546.json
```

#### 自定义文件名

```bash
trae run "修复main.py中的bug" --trajectory-file my_debug_session.json
# 轨迹保存至：my_debug_session.json
```

#### 交互模式

```bash
trae interactive --trajectory-file session.json
```

### 编程方式使用

```python
from trae_agent.agent.trae_agent import TraeAgent
from trae_agent.utils.llm_client import LLMProvider
from trae_agent.utils.config import ModelParameters

# 创建代理
agent = TraeAgent(LLMProvider.ANTHROPIC, model_parameters, max_steps=10)

# 设置轨迹记录
trajectory_path = agent.setup_trajectory_recording("my_trajectory.json")

# 配置并运行任务
agent.new_task("我的任务", task_args)
execution = await agent.execute_task()

# 轨迹自动保存
print(f"轨迹保存至：{trajectory_path}")
```

## 轨迹文件格式

轨迹文件是一个JSON文档，具有以下结构：

```json
{
  "task": "任务描述",
  "start_time": "2025-06-12T22:05:46.433797",
  "end_time": "2025-06-12T22:06:15.123456",
  "provider": "anthropic",
  "model": "claude-sonnet-4-20250514",
  "max_steps": 20,
  "llm_interactions": [
    {
      "timestamp": "2025-06-12T22:05:47.000000",
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514",
      "input_messages": [
        {
          "role": "system",
          "content": "你是一名软件工程助手..."
        },
        {
          "role": "user",
          "content": "创建一个hello world Python脚本"
        }
      ],
      "response": {
        "content": "我将帮你创建一个hello world Python脚本...",
        "model": "claude-sonnet-4-20250514",
        "finish_reason": "end_turn",
        "usage": {
          "input_tokens": 150,
          "output_tokens": 75,
          "cache_creation_input_tokens": 0,
          "cache_read_input_tokens": 0,
          "reasoning_tokens": null
        },
        "tool_calls": [
          {
            "call_id": "call_123",
            "name": "str_replace_based_edit_tool",
            "arguments": {
              "command": "create",
              "path": "hello.py",
              "file_text": "print('Hello, World!')"
            }
          }
        ]
      },
      "tools_available": ["str_replace_based_edit_tool", "bash", "task_done"]
    }
  ],
  "agent_steps": [
    {
      "step_number": 1,
      "timestamp": "2025-06-12T22:05:47.500000",
      "state": "thinking",
      "llm_messages": [...],
      "llm_response": {...},
      "tool_calls": [
        {
          "call_id": "call_123",
          "name": "str_replace_based_edit_tool",
          "arguments": {...}
        }
      ],
      "tool_results": [
        {
          "call_id": "call_123",
          "success": true,
          "result": "文件创建成功",
          "error": null
        }
      ],
      "reflection": null,
      "error": null
    }
  ],
  "success": true,
  "final_result": "Hello world Python脚本创建成功！",
  "execution_time": 28.689999
}
```

### 字段说明

**根级别：**

- `task`：原始任务描述
- `start_time`/`end_time`：ISO格式时间戳
- `provider`：使用的LLM提供商（例如："anthropic"、"openai"、"google"、"azure"、"doubao"、"ollama"、"openrouter"）
- `model`：模型名称
- `max_steps`：允许的最大执行步骤
- `success`：任务是否成功完成
- `final_result`：最终输出或结果消息
- `execution_time`：总执行时间（秒）

**LLM交互：**

- `timestamp`：交互发生的时间
- `provider`：本次交互使用的LLM提供商
- `model`：本次交互使用的模型
- `input_messages`：发送给LLM的消息
- `response`：完整的LLM响应，包括内容、使用情况和工具调用
- `tools_available`：本次交互期间可用的工具列表

**代理步骤：**

- `step_number`：顺序步骤编号
- `state`：代理状态（"thinking"、"calling_tool"、"reflecting"、"completed"、"error"）
- `llm_messages`：本步骤中使用的消息
- `llm_response`：本步骤的LLM响应
- `tool_calls`：本步骤中调用的工具
- `tool_results`：工具执行的结果
- `reflection`：代理对步骤的反思
- `error`：如果步骤失败，包含错误消息

## 优势

1. **调试**：精确跟踪代理执行期间发生的情况
2. **分析**：了解LLM推理和工具使用模式
3. **审计**：维护有关所做更改及其原因的记录
4. **研究**：分析代理行为以进行改进
5. **合规性**：保留自动化操作的详细日志

## 文件管理

- 轨迹文件默认保存在当前工作目录中
- 如果未提供自定义路径，文件使用基于时间戳的命名
- 文件会自动创建/覆盖
- 系统会在需要时处理目录创建
- 文件在执行过程中持续保存（不仅仅在结束时）

## 安全考虑

- 轨迹文件可能包含敏感信息（API密钥不会被记录）
- 如果轨迹文件包含专有代码或数据，请安全存储
- 轨迹文件会自动保存到`trajectories/`目录，该目录已被排除在版本控制之外

## 示例用例

1. **调试失败任务**：查看代理执行中哪里出错了
2. **性能分析**：分析令牌使用和执行模式
3. **合规审计**：跟踪代理所做的所有更改
4. **模型比较**：比较不同LLM提供商/模型的行为
5. **工具使用分析**：了解哪些工具被使用以及使用频率