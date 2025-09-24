感谢您对 Trae Agent 项目的关注和贡献！我们欢迎社区的各种贡献。

## 贡献方式

您可以通过多种方式为 Trae Agent 做出贡献：

- **代码贡献**：添加新功能、修复 bug 或提高性能
- **文档**：改进 README、添加代码注释或创建示例
- **bug 报告**：通过 issues 提交详细的 bug 报告
- **功能请求**：建议新功能或改进
- **代码审查**：审查其他贡献者的拉取请求
- **社区支持**：在讨论和 issues 中帮助他人

## 开发环境设置

1. Fork 代码仓库
2. 克隆您的 fork：

   ```bash
   git clone https://github.com/bytedance/trae-agent.git
   cd trae-agent
   ```

3. 设置您的开发环境：

   ```bash
   make install-dev
   make pre-commit-install
   ```

## 运行测试

```bash
make test
```

## 开发流程

1. 创建一个新分支：

   ```bash
   git checkout -b feature/amazing-feature
   ```

2. 按照我们的编码标准进行更改：
   - 编写清晰、有文档的代码
   - 遵循 PEP 8 风格指南
   - 为新功能添加测试
   - 必要时更新文档
   - 维护类型提示，并在可能的情况下添加类型检查

3. 提交您的更改：

   ```bash
   git commit -m 'Add some amazing feature'
   ```

4. 推送到您的 fork：

   ```bash
   git push origin feature/amazing-feature
   ```

5. 打开一个拉取请求

## 拉取请求指南

- 完整填写拉取请求模板
- 为新功能添加测试
- 必要时更新文档
- 确保所有测试通过，没有 linting 错误
- 保持拉取请求专注于单一功能或修复
- 引用任何相关的 issues

## 代码风格

- 遵循 PEP 8 指南
- 尽可能使用类型提示
- 编写描述性的文档字符串
- 保持函数和方法专注于单一目的
- 对复杂逻辑添加注释
- Python 版本要求：>= 3.12

## 社区指南

- 保持尊重和包容
- 遵循我们的行为准则
- 帮助他人学习和成长
- 提供建设性的反馈
- 专注于改进项目

## 需要帮助？

如果您在任何方面需要帮助：

- 查看现有的 issues 和讨论
- 加入我们的社区渠道
- 在讨论中提问

## 许可证

通过为 Trae Agent 做出贡献，您同意您的贡献将在 MIT 许可证下获得许可。

我们感谢您为使 Trae Agent 变得更好所做的贡献！