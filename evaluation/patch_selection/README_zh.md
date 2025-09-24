此文档介绍如何使用选择器代理（selector agent）进一步增强 [Trae Agent](https://github.com/bytedance/trae-agent)。
选择器代理是首个基于代理的集成推理方法，用于仓库级问题解决。
它将我们的目标表述为最优解决方案搜索问题，并通过用于生成、剪枝和选择的模块化代理来解决两个关键挑战，即大型集成空间和仓库级理解。

## 📖 演示

### 回归测试
对于回归测试，请参考 [Agentless](https://github.com/OpenAutoCoder/Agentless/blob/main/README_swebench.md)。

每个结果条目包含一个 `regression` 字段，指示测试结果：
   - 空数组 [] 表示补丁成功通过了所有回归测试；
   - 任何非空值表示补丁导致测试失败（详细说明哪些测试失败）。

### 准备工作

**重要提示：** 您需要从 [Google Drive](https://drive.google.com/file/d/1dF7kbcmdLRJu7TEh8G7Oe8_6NY3aieKa/view?usp=sharing) 下载 Python 3.12 包，并将其解压缩到 `evaluation/patch_selection/trae_selector/tools/py312`。这用于在 Docker 容器中执行代理工具。

### 输入格式

补丁候选存储在 JSON 行文件中。对于每个实例，结构如下：

```json
{
    "instance_id": "django__django-14017",
    "issue": "问题描述...",
    "patches": [
        "补丁差异 1",
        "补丁差异 2",
        ...,
        "补丁差异 N",
    ],
    "success_id": [
        1,
        0,
        ...,
        1
    ],
    "regressions": [
      [补丁差异 1 的回归测试名称...],
      [补丁差异 2 的回归测试名称...],
      ...,
      [补丁差异 N 的回归测试名称...],
    ]
}
```

注意：success_id 为 1（对应的补丁差异是正确的补丁）或 0（对应的补丁差异是错误的补丁）。一旦选择器代理选择了一个补丁，我们可以快速报告所选补丁是否正确。

regressions 字段是可选的。如果您已经使用 Agentless 完成了回归测试选择，可以在此处填写所选的回归测试。

### 补丁选择

```bash
python3 evaluation/patch_selection/selector.py \
    --instances_path "path/to/swebench-verified.json" \
    --candidate_path "path/to/patch_candidates.jsonl" \
    --result_path "path/to/save/results" \
    --num_candidate 每个实例的补丁候选数量 \
    --max_workers 10 \
    --group_size 组大小 \
    --max_retry 20 \
    --max_turn 200 \
    --config_file trae_config.yaml \
    --model_name 配置文件中的模型名称 \
    --majority_voting
```

注意：如果您有很多补丁候选，例如 50 个，可以将 group_size 设置为 10。补丁选择通过 5（50/10）个组进行。每个组选择一个补丁。然后您可以从这 5 个中选择。

`--majority_voting` 是可选的。如果启用，对于每个候选组，进行多次补丁选择，选择频率最高的补丁是最终答案。此模式会消耗更多的令牌。

### 示例

使用 [example.jsonl](example/example.jsonl) 运行后，在 result_path 中，我们得到以下文件：

```text
├── log
│   └── group_0
│       └── astropy__astropy-14369_voting_0_trail_1.json
├── output
│   └── group_0
│       └── astropy__astropy-14369.log
├── patch
│   └── group_0
│       └── astropy__astropy-14369_1.patch
└── statistics
    └── group_0
        └── astropy__astropy-14369.json
```

* log 目录中的文件存储 LLM 交互历史。
* output 目录中的文件存储原始标准输出和标准错误。
* patch 目录存储选定的补丁。
* statistics 目录存储选定的补丁是否正确。

您可以使用 `analysis.py` 脚本可视化选择结果（即使在选择运行期间也可以查看中间结果）

```bash
python3 analysis.py --output_path "path/to/save/results"
```