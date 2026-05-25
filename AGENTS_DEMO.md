# 代码生成与修改规范
1. 专注当前任务：仅修改与当前需求直接相关的代码，严禁改动其他已正常工作的功能模块。
2. 增量修改：修改函数前先理解原逻辑，保留原有核心逻辑，禁止盲目重构。
3. 代码风格：代码中禁止包含任何 Emoji 表情；不轻易注释，用有意义的变量名和函数名代替。
4. 简化逻辑：专注于主流程实现，忽略 Fallback 降级处理；除非明确要求，否则不生成测试脚本和项目文档。
5. 环境约束：基于 Linux/Windows/Mac(自己修改) 环境开发，确保所有文件使用 UTF-8 编码，杜绝中文乱码。
6. Markdown 文档一律使用中文；Git commit message 一律使用英文单句，并遵循 AngularJS/Conventional Commits 格式，具体见下，不得添加正文、脚注或协作者信息；严禁出现 `Claude`、`ChatGPT`、`AI`、`Co-Authored-By`、`Generated with` 等任何表明由大模型生成或协作的内容。
```
<type>(<scope>): <summary>
<正文：描述本次变更的背景与动机>
Agent-Task: <原始任务描述或任务 ID>
Agent-Decision: <关键设计决策及理由>
Agent-Limitation: <已知局限或后续 TODO>
```
7. 遇到运行、构建或测试错误时，将错误摘要、必要复现命令、测试信息和环境信息，以一句话追加到项目的 AGENTS.md 或 CLAUDE.md。
8. 使用uv管理和运行python项目。
