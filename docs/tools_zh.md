# 工具

Trae Agent 提供了五种内置工具用于软件工程任务：

## str_replace_based_edit_tool

具有持久状态的文件和目录操作工具。

**操作：**
- `view` - 显示带行号的文件内容，或列出最多2级深度的目录内容
- `create` - 创建新文件（如果文件已存在则失败）
- `str_replace` - 替换文件中的精确字符串匹配（必须唯一）
- `insert` - 在指定行号后插入文本

**主要特点：**
- 需要绝对路径（例如，`/repo/file.py`）
- 字符串替换必须完全匹配，包括空白字符
- 支持大文件的行范围查看

## bash

在持久会话中执行 shell 命令。

**特点：**
- 命令在共享的 bash 会话中运行，保持状态
- 每个命令 120 秒超时
- 会话重启功能
- 后台进程支持

**使用注意事项：**
- 使用 `restart: true` 重置会话
- 避免输出过多的命令
- 长时间运行的命令应使用 `&` 进行后台执行

## sequential_thinking

用于复杂分析的结构化问题解决工具。

**功能：**
- 将问题分解为顺序思考步骤
- 修改和分支之前的思考
- 动态调整所需思考数量
- 跟踪思考历史和替代方法
- 生成和验证解决方案假设

**参数：**
- `thought` - 当前思考步骤
- `thought_number` / `total_thoughts` - 进度跟踪
- `next_thought_needed` - 继续思考标志
- `is_revision` / `revises_thought` - 修改跟踪
- `branch_from_thought` / `branch_id` - 替代方案探索

## task_done

带有验证要求的任务完成信号。

**目的：**
- 将任务标记为成功完成
- 只能在适当验证后调用
- 鼓励编写测试/复现脚本

**输出：**
- 简单的 "Task done." 消息
- 不需要参数

## json_edit_tool

使用 JSONPath 表达式精确编辑 JSON 文件。

**操作：**
- `view` - 显示整个文件或特定 JSONPath 处的内容
- `set` - 更新指定路径处的现有值
- `add` - 向对象添加新属性或向数组追加元素
- `remove` - 删除指定路径处的元素

**JSONPath 示例：**
- `$.users[0].name` - 第一个用户的名称
- `$.config.database.host` - 嵌套对象属性
- `$.items[*].price` - 所有项目的价格
- `$..key` - 递归搜索关键字

**特点：**
- 验证 JSON 语法和结构
- 保留格式化，可选漂亮打印
- 为无效操作提供详细错误信息