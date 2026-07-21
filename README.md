# 项目指南技能

这是一个 Codex 技能，用于分析陌生软件项目并生成面向新成员的项目指南。它强调先观察、再判断，并把关键结论绑定到文件路径、配置、命令输出或测试结果等证据上。

## 适用场景

- 快速理解一个新仓库的用途、技术栈和目录结构
- 梳理启动、构建、测试和部署路径
- 追踪核心数据流或控制流
- 为团队成员编写入门文档、交接文档或架构说明
- 对认证、权限、配置、依赖等风险面做轻量审查

## 项目结构

```text
.
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── analysis-checklist.md
    ├── frontend-projects.md
    ├── output-templates.md
    └── security-review.md
```

## 文件说明

| 路径 | 作用 |
| --- | --- |
| `SKILL.md` | 技能主说明，定义使用场景、分析流程、输出要求和完成标准。 |
| `agents/openai.yaml` | Codex 界面展示信息和默认提示词。 |
| `references/analysis-checklist.md` | 通用仓库分析检查清单。 |
| `references/frontend-projects.md` | 前端项目专项分析问题、流程和验证建议。 |
| `references/output-templates.md` | 项目指南、快速分析、技术交接的输出模板。 |
| `references/security-review.md` | 安全、认证、权限和密钥处理相关审查清单。 |

## 简单分析

该项目不是传统运行型应用，而是一个面向 Codex 的技能包。它的核心价值在于把项目分析过程标准化：先做只读发现，识别技术栈和入口，再追踪核心流程，最后输出带证据、风险和下一步建议的中文报告。

当前内容以文档和提示词为主，没有构建脚本、测试脚本或运行时依赖。主要维护点是 `SKILL.md` 和 `references/` 下的清单模板。后续如果继续扩展，可以补充更多专项参考，例如后端服务、CLI 工具、数据项目或 DevOps 项目的分析清单。

## 使用方式

在 Codex 中调用：

```text
使用 $project-guide-skill 分析该仓库，生成面向新成员的项目指南。
```

也可以指定分析深度：

```text
使用 $project-guide-skill 对该仓库做 deep 分析，重点追踪核心请求流程和测试覆盖缺口。
```
