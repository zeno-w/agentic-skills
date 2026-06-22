# Soil Agentic Java Spec

Java 服务端 AI Coding 规范 Skill 工程。通过 git submodule 引入到 Java 服务端项目的 `.trae/skills/` 目录，为 AI 编程助手提供 Java 开发规范上下文。

## 使用方式

在目标 Java 项目中，将本仓库作为 submodule 添加到 skill 目录：

```bash
# 添加 submodule
git submodule add <repo-url> .trae/skills/java-spec

# 初始化并拉取（已有项目克隆后）
git submodule update --init --recursive
```

添加后，Trae 等 AI IDE 会自动加载 `.trae/skills/` 下的所有 SKILL.md，在编码时提供规范指导。

## Skill 清单

| Skill | 说明 | 触发场景 |
|-------|------|---------|
| `java-coding-guidelines` | 阿里巴巴 Java 开发手册规约 | 编写/审查 Java 代码、PR Review、代码风格修复 |
| `mysql-conventions` | MySQL 数据库设计、SQL 编写、ORM 规范 | 设计表结构、编写 SQL、使用 MyBatis/JPA |

## 目录结构

```
soil-agentic-java-spec/
├── java-coding-guidelines/       # 阿里 Java 编码规范
│   ├── SKILL.md                          # 入口：导航枢纽
│   └── references/                       # 详细规则
│       ├── naming-conventions.md         # 命名规约
│       ├── oop-conventions.md            # OOP 规约
│       ├── collection-concurrency.md     # 集合与并发
│       ├── exception-logging-control.md  # 异常、日志与控制
│       ├── format-comments-structure.md  # 格式、注释与工程结构
│       └── unit-test-security.md         # 单元测试与安全
├── mysql-conventions/                    # MySQL 数据库规范
│   ├── SKILL.md                          # 入口：导航枢纽
│   └── references/                       # 详细规则
│       ├── table-design.md               # 表设计
│       ├── index-rules.md                # 索引规则
│       ├── sql-rules.md                  # SQL 编写
│       └── orm-rules.md                  # ORM（MyBatis/JPA）
└── skills-lock.json                      # 外部 skill 锁定文件
```

## Skill 设计原则

- **导航枢纽**：SKILL.md 只做路由和流程指引，不重复展开规则细节
- **单一信息源**：规则详细内容只在 `references/` 中维护，修改一处即可
- **按需加载**：模型根据导航表的 "When to Read" 列，按场景读取对应 reference 文件

## 新增 Skill

参考已有 skill 的结构：

```
<skill-name>/
├── SKILL.md          # frontmatter (name + description) + 导航表 + 使用流程
└── references/       # 详细规则文档
    └── *.md
```

SKILL.md 的 frontmatter 格式：

```yaml
---
name: "<skill-name>"
description: "<功能说明>. Invoke when <触发条件>."
---
```