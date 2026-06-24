---
name: "java-sdk-doc-generator"
description: "Generates AI-agent-friendly Markdown SDK docs from Java source comments. Detects changes via git status and auto-updates docs. Invoke when user asks to generate/update SDK docs, Java API docs, or mentions 'generate SDK documentation'."
---

# Soil SDK Doc Generator

将 Java 源码解析为 AI Agent 友好的 Markdown SDK 文档，支持基于 git 状态的增量更新。

## 执行步骤

| 阶段                 | 步骤                          | 参考文件 |
|--------------------|-----------------------------|----------|
| 分析                 | 确定范围 → 检测 Git 变更 → 解析 Java 源码 | [analyze.md](analyze.md) |
| 检查 rules submodule | 执行脚本同步 rules submodule | [sync-agent-rules.sh](sync-agent-rules.sh) |
| 产出                 | 生成 Markdown → 输出文件 → 汇总     | [produce.md](produce.md) |
| 提交发布              | 检测变更 → 提交 → 打标签 → 推送 submodule | [commit.md](commit.md) |
| 模板                 | Markdown 输出格式               | [template.md](template.md) |

> 每个阶段执行前，先读取对应的参考文件获取详细规则。
> 「提交发布」阶段在「产出」完成后自动执行，若 submodule 有增删改则提交、打标签并推送。