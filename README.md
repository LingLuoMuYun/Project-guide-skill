# Project Guide Skill

让 AI 帮你**快速搞懂任何代码项目**——分析仓库结构、追踪核心流程、梳理埋点事件，输出 Markdown 报告或交互式 HTML 指南。

**跨平台**：支持 Claude Code、Codex、Cursor、ChatGPT、Gemini 等任何 AI Agent。

---

## 三种分析模式

激活技能后，AI 会先让你选择：

| 模式 | 产物 | 一句话 |
| --- | --- | --- |
| 📄 **浅层分析** | `project-guide/project-guide.md` | 快速 Markdown 报告：技术栈、结构、流程概要 |
| 🌐 **深层分析** ⭐ | `project-guide/` 目录（约 17 个文件） | 交互式 HTML 指南 + 业务代码逐行分析 + 模块化架构 |
| 🔍 **聚焦分析** | `project-guide/{模块名}-focus.md` 或 `.html` | 指定模块/功能深入剖析，逐文件逐行 |

也可以一步到位：`聚焦分析 用户认证模块，输出 html`

---

## 核心能力

| 能力 | 说明 |
| --- | --- |
| 🗂️ **项目结构树** | 带注释的完整目录树，P0-P3 优先级标注、依赖方向图、循环依赖识别 |
| 📝 **代码分析** | P0/P1 文件摘要卡片 + 关键代码逐行分析（深层/聚焦模式） |
| 📊 **埋点梳理** | SDK 识别、事件清单、覆盖缺口检查 |
| 🧭 **快速导航** | 按模块/文件类型/常用命令三维索引 |
| 🌐 **HTML 指南** | 模块化架构：12 个 Section 独立渲染、搜索、暗/亮主题、Mermaid 图、快捷键 |

---

## 触发关键词

无需记命令，自然语言即可触发。以下是几类典型说法：

```text
# 快速入门
帮我快速入手这个项目    分析下这个仓库的结构

# 指引生成
生成一份项目指南        给我一份入门指引

# 梳理分析
梳理下项目的埋点        讲讲这个项目怎么运作的

# 交接
我接手这个项目，帮我理清结构

# 聚焦分析（指定模块深入）
聚焦分析 用户认证模块，输出 html
针对 src/services/order/ 做深入分析
只看支付流程，帮我理清楚
```

完整关键词覆盖：`快速入手` `项目分析` `代码解析` `架构解析` `结构梳理` `目录树` `埋点梳理` `项目交接` `交接文档` `project guide` `onboarding` `focus on` `deep dive into` 等。

---

## 使用方式

### Claude Code（Skill 机制）

```bash
git clone <本仓库> .claude/skills/project-guide-skill
```

然后直接说：

```text
/project-guide-skill 分析当前项目
帮我快速入手这个项目
聚焦分析 支付模块
```

自然语言触发，无需记住命令名。

### 任何 AI Agent（通用方式）⭐

1. 打开 [PROMPT.md](PROMPT.md)，复制 `---` 分隔线之间的内容
2. 粘贴到 Claude / ChatGPT / Gemini / Kimi / 通义千问等任意对话
3. 告诉 AI 项目路径：

```text
请分析 /path/to/my-project 这个项目。
```

### 聚焦分析示例

```text
# 一步到位
聚焦分析 用户认证模块，输出 html

# 或先触发 skill，再从菜单中选择"选项 3"
帮我快速入手这个项目
→ AI 展示三个选项 → 选 3 → 告诉 AI 要聚焦的模块
```

---

## 项目结构

```text
.
├── SKILL.md                      # 技能主说明（skill 平台加载入口）
├── PROMPT.md                     # ★ 通用 prompt（自包含，可复制到任何 AI Agent）
├── README.md
├── agents/
│   └── openai.yaml               # Codex 界面配置
└── references/
    ├── analysis-checklist.md      # 分析检查清单
    ├── output-templates.md        # MD / HTML 输出模板
    ├── project-structure.md       # 结构树分析与代码摘要规范
    ├── tracking-analytics.md      # 埋点与追踪分析方法
    ├── html-guide-template.md     # 模块化 HTML 架构：核心框架、Section 模板、CSS、扩展指南
    ├── project-types.md           # 项目形态专项：前端/后端/Monorepo/安全审查
    └── large-projects.md          # 大型项目优化：规模分级、采样策略、加速技巧
```

---

## 关于本仓库

这是一个**跨平台 AI Agent 技能包**——不是传统应用。核心价值是将项目分析过程标准化：

```
只读盘点 → 识别技术栈 → 追踪核心流程 → 梳理结构树
→ 分析关键代码 → 识别埋点 → 输出报告/HTML 指南
```

所有结论绑定证据（文件路径、配置、命令输出）。当前维护重点是 `SKILL.md`、`PROMPT.md` 和 `references/` 下的模板文档。

---

## License

MIT
