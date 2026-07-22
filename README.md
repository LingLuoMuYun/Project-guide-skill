# 项目指南技能

这是一个**跨平台 AI Agent 技能**，用于**快速接手陌生项目**——分析软件项目并生成面向新成员的项目指南。它强调先观察、再判断，并把关键结论绑定到文件路径、配置、命令输出或测试结果等证据上。支持 Claude Code、Codex、Cursor、ChatGPT、Gemini 等任何 AI Agent。

**两种分析模式，按需选择：**

| 模式 | 产物 | 特点 |
| --- | --- | --- |
| 📄 **浅层分析** | `project-guide/project-guide.md` | 快速生成 Markdown 报告，适合快速了解项目概况 |
| 🌐 **深层分析** ⭐ | `project-guide/index.html` | 交互式 HTML 指南，**含关键业务代码展示与逐行分析**、Mermaid 架构图、可搜索埋点表格、主题切换 |

激活技能后，AI 会先让你选择浅层还是深层模式，无需提前决定。

## 触发关键词

在对话中使用以下任何词汇即可触发此技能，无需显式调用 skill 名称：

**快速入门：** `快速入手` `快速上手` `快速了解` `快速熟悉` `认识项目` `看懂项目` `入门`
**指引指南：** `项目指引` `项目指南` `入门指南` `生成项目指南` `项目导航`
**解析分析：** `项目解析` `代码解析` `架构解析` `项目分析` `代码分析` `项目解读` `代码解读`
**梳理整理：** `梳理项目` `结构梳理` `项目梳理` `目录树` `结构树` `项目概览` `项目总览`
**接手交接：** `接手项目` `项目交接` `交接文档` `熟悉项目` `了解项目`
**埋点追踪：** `埋点梳理` `埋点分析` `追踪分析` `埋点`
**英文触发：** `project guide` `onboarding` `analyze this project` `understand this project`

典型说法：
- "帮我**快速入手**这个项目"
- "**分析**下这个仓库的结构"
- "**梳理**一下这个项目的**埋点**"
- "给我一份这个项目的**入门指引**"
- "**讲讲**这个项目是怎么运作的"

## 适用场景

- 快速理解一个新仓库的用途、技术栈和目录结构
- 梳理启动、构建、测试和部署路径
- **生成带注释的项目结构树（含优先级标注和依赖方向）**
- **分析关键代码文件的内容、职责和调用关系**
- **识别和整理项目中的埋点事件（tracking/analytics），发现覆盖缺口**
- 追踪核心数据流或控制流
- **构建快速导航索引（按模块、文件类型、常用命令）**
- **生成交互式 HTML 项目指南（离线可用，含搜索、主题切换、目录树）**
- 为团队成员编写入门文档、交接文档或架构说明
- 对认证、权限、配置、依赖等风险面做轻量审查

## 项目结构

```text
.
├── SKILL.md                      # 技能主说明（适合有 skill 机制的平台）
├── PROMPT.md                     # ★ 通用 prompt（适合在任何 AI Agent 中复制使用）
├── README.md
├── agents/
│   └── openai.yaml               # Codex 界面配置（Codex 专属）
└── references/
    ├── analysis-checklist.md      # 通用仓库分析检查清单
    ├── output-templates.md        # Markdown 和 HTML 输出模板
    ├── project-structure.md       # 项目结构树分析和代码文件摘要规范
    ├── tracking-analytics.md      # 埋点与追踪分析方法和清单模板
    ├── html-guide-template.md     # HTML 项目指南模板和 CSS 规范
    ├── frontend-projects.md       # 前端项目专项分析
    └── security-review.md         # 安全、认证、权限审查清单
```

## 文件说明

| 路径 | 作用 |
| --- | --- |
| `SKILL.md` | 技能主说明，定义使用场景、分析流程、输出要求和完成标准。在 Claude Code / Codex 等有 skill 机制的平台上直接作为 skill 加载。 |
| `PROMPT.md` | ★ **通用 prompt**：自包含的完整分析指令，可复制粘贴到任何 AI Agent（ChatGPT、Claude、Gemini、Kimi 等）中使用，不依赖 reference 文件。 |
| `agents/openai.yaml` | Codex 界面展示信息和默认提示词（仅 Codex 平台需要）。 |
| `references/analysis-checklist.md` | 通用仓库分析检查清单（含结构树、代码分析、埋点、导航）。 |
| `references/output-templates.md` | Markdown 项目指南、快速分析、技术交接及 HTML 指南的输出模板。 |
| `references/project-structure.md` | 项目结构树分析规范、代码文件摘要模板、依赖关系图和快速导航索引。 |
| `references/tracking-analytics.md` | 埋点 SDK 识别、调用模式搜索、埋点清单模板和覆盖检查清单。 |
| `references/html-guide-template.md` | HTML 指南的完整样板：布局、CSS 变量、交互功能、响应式设计。 |
| `references/frontend-projects.md` | 前端项目专项分析问题、流程和验证建议。 |
| `references/security-review.md` | 安全、认证、权限和密钥处理相关审查清单。 |

## 核心能力

### 1. 项目结构树梳理
生成带注释的完整目录树，标注优先级（P0-P3）、特殊标记（入口 ★ / 核心 ● / 配置 ▲ / 风险 ⚠）、依赖方向，识别循环依赖和"上帝模块"。

### 2. 代码文件内容分析
对关键文件生成结构化摘要卡片：职责、核心导出、依赖关系、关键逻辑、潜在问题、修改影响范围。

### 3. 埋点与追踪识别
搜索埋点 SDK 引用和调用模式，整理埋点事件清单（事件名、触发位置、参数），发现覆盖缺口，检查合规性。

### 4. 快速导航
按功能模块、文件类型、常用命令三个维度构建跳转索引，让新人能快速定位到所需内容。

### 5. HTML 项目指南
在目标项目根目录生成 `project-guide/index.html`，一个自包含的交互式 HTML 页面：
- 左侧固定导航 + 右侧内容区 + 顶部搜索
- 可展开/折叠的目录树
- Mermaid 架构图和时序图
- 可搜索的埋点清单表格
- 暗色/亮色主题切换
- 响应式设计（桌面/平板/手机）
- 键盘快捷键

## 简单分析

该项目不是传统运行型应用，而是一个**跨平台 AI Agent 技能包**。它的核心价值在于把项目分析过程标准化：先做只读发现，识别技术栈和入口，再追踪核心流程，梳理项目结构树，分析关键代码文件，识别埋点和追踪，最后输出带证据、风险、快速导航和下一步建议的中文报告，并生成交互式 HTML 项目指南。

当前内容以文档和提示词为主，没有构建脚本、测试脚本或运行时依赖。主要维护点是 `SKILL.md`、`PROMPT.md` 和 `references/` 下的清单模板。

## 使用方式

### 在 Claude Code 中（Skill 机制）

克隆本仓库到 `.claude/skills/` 目录（或通过设置配置 skill 路径）。**支持自然语言触发**，以下说法都能激活此 skill：

```text
/project-guide-skill 分析当前项目
帮我快速入手这个项目
梳理一下这个项目的结构和埋点
这个项目是怎么回事，给我讲讲
生成一份项目指南
分析下这个仓库的架构
```

无需刻意记住 `/project-guide-skill` 命令，说出"分析项目""快速入手""梳理结构"等关键词即可触发。

### 在 Codex 中

```text
使用 $project-guide-skill 分析该仓库，生成面向新成员的项目指南。
```

指定分析深度：

```text
使用 $project-guide-skill 对该仓库做 deep 分析，重点追踪核心请求流程和测试覆盖缺口。
```

### 在任何 AI Agent 中（通用方式）⭐

1. 打开 [PROMPT.md](PROMPT.md)，复制 `---` 分隔线之间的全部内容
2. 粘贴到任意 AI 对话中（Claude、ChatGPT、Cursor、Gemini、Kimi、通义千问等）
3. 告诉 AI 你的项目路径：

```text
请分析 /path/to/my-project 这个项目。
```

### 聚焦特定需求

```text
# Claude Code
/project-guide-skill 分析该前端项目，重点梳理项目结构树、识别埋点和生成 HTML 指南。

# 通用方式（在任何 Agent 中）
[粘贴 PROMPT.md 内容后]
请分析这个 Vue3 后台管理项目，只关注项目结构树和埋点识别，不需要生成 HTML 指南。
```
