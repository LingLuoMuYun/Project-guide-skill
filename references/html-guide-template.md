# HTML 项目指南模板

当 skill 完成分析后，在目标项目根目录创建 `project-guide/index.html`，生成一个自包含的交互式 HTML 指南文件。

## 生成要求

### 基本要求

- **自包含**：所有 CSS/JS/Font 内联，不依赖外部文件（Mermaid 可通过 CDN 加载）
- **单文件**：一个 `index.html` 即可在浏览器中打开
- **离线可用**：除了 Mermaid CDN（可选），其他资源全部内联
- **文件位置**：`{目标项目根目录}/project-guide/index.html`

### 设计原则

- 左侧固定导航 + 右侧内容区（经典文档布局）
- 左侧导航包含可展开的树形目录
- 顶部搜索栏可过滤模块和文件
- 暗色/亮色主题切换按钮
- 响应式设计，移动端导航收缩为汉堡菜单
- Mermaid 图表渲染核心流程和架构

## HTML 模板结构

```html
<!DOCTYPE html>
<html lang="zh-CN" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{项目名称} - 项目指南</title>
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <style>/* 内联全部 CSS */</style>
</head>
<body>
  <!-- 顶部工具栏 -->
  <header class="top-bar">
    <button class="menu-toggle" id="menuToggle">☰</button>
    <h1>{项目名称} 项目指南</h1>
    <div class="header-tools">
      <input type="text" class="search-input" id="searchInput" placeholder="搜索模块、文件、函数...">
      <button class="theme-toggle" id="themeToggle">🌓</button>
    </div>
  </header>

  <!-- 左侧导航 -->
  <nav class="sidebar" id="sidebar">
    <div class="nav-section">
      <div class="nav-title">📋 快速导航</div>
      <!-- 由分析结果动态填充 -->
    </div>
    <div class="nav-section">
      <div class="nav-title">📁 项目结构</div>
      <!-- 可展开的目录树 -->
    </div>
    <div class="nav-section">
      <div class="nav-title">📊 埋点清单</div>
      <!-- 埋点事件链接 -->
    </div>
  </nav>

  <!-- 遮罩层（移动端） -->
  <div class="overlay" id="overlay"></div>

  <!-- 主内容区 -->
  <main class="content">
    <!-- 各章节由分析结果动态填充 -->
  </main>

  <script>/* 内联全部 JS */</script>
</body>
</html>
```

## 内容章节

HTML 指南应包含以下章节（根据实际分析结果增减）：

### 1. 项目概览
```
┌─────────────────────────────────┐
│ 🏷️ 项目名称                    │
│ 📝 一句话描述                   │
│ 👥 目标用户                     │
│ 🚀 当前成熟度                   │
│ 🏗️ 技术栈标签云                 │
└─────────────────────────────────┘
```
- 使用卡片式布局展示基本信息
- 技术栈以彩色标签展示

### 2. 快速开始
- 环境要求
- 安装命令
- 启动命令
- 访问地址
- 一键复制按钮

### 3. 项目结构树
- 交互式目录树（可展开/折叠）
- 每个节点有图标和简要说明
- 关键文件高亮标注
- 支持按层级展开（全部展开 / 全部折叠 / 展开到第 N 层）

### 4. 架构概览
- Mermaid 图表：系统架构图
- Mermaid 图表：核心流程图
- 分层说明

### 5. 代码展示与分析 ⭐（深层模式核心章节）
这是深层 HTML 指南区别于浅层 Markdown 报告的关键差异点。每段代码使用独立的展示面板：

#### 面板结构
```
┌─────────────────────────────────────────────┐
│ 📄 文件路径:行号范围                  [复制] │ ← 代码头部（文件路径 + 复制按钮）
├─────────────────────────────────────────────┤
│  1 │ // 实际项目代码                       │ ← 代码内容区（带行号）
│  2 │ export function login(params) {       │
│  3 │   const res = await request.post(    │
│  4 │     '/api/auth/login', params        │
│  5 │   );                                  │
│  6 │   if (res.code === 0) {              │
│  7 │     setToken(res.data.token);         │
│  8 │     userStore.setUser(res.data.user); │
│  9 │   }                                   │
│ 10 │   return res;                         │
│ 11 │ }                                      │
├─────────────────────────────────────────────┤
│ 🔍 代码分析                                │ ← 分析注释区（浅蓝背景区分）
│                                             │
│ 功能：发送登录请求，成功后存储 token 并    │
│       更新全局用户状态                      │
│                                             │
│ 逐行说明：                                  │
│ · L1: 异步函数声明，接收登录凭据参数       │
│ · L2-4: 调用封装的 HTTP 实例发送 POST      │
│ · L5-8: 响应码为 0 表示成功，执行两项操作  │
│   → setToken：将 token 写入持久化存储      │
│   → userStore.setUser：更新全局状态        │
│   ⚠ 注意：两个操作无事务保证，需关注顺序   │
│ · L10: 返回完整响应给上层调用方            │
│                                             │
│ 输入：{ username, password, captcha? }     │
│ 输出：{ code, data: { token, user }, msg } │
│ 调用方：LoginPage.vue, AuthModal.vue       │
│ ⚠ 注意：未处理网络超时和断网情况           │
└─────────────────────────────────────────────┘
```

#### 代码提取规则

**必须提取的代码类型：**
- 入口文件初始化逻辑（应用创建、插件注册、全局配置）
- 路由表定义（完整路由配置）
- API 服务层（2-5 个核心接口实现）
- 状态管理（Store/Context 的核心结构定义）
- 埋点调用（实际 track/logEvent 调用代码）
- 核心业务流程（1-3 个关键函数/组件）
- 复杂或易出错的逻辑段

**每段代码必须附带的分析：**
- 功能概述（一句话）
- 逐行/逐段说明（关键行的解释）
- 输入/输出（类型和含义）
- 调用方列表（谁在使用这段代码）
- 注意事项（边界条件、已知问题、性能考量）
- ⚠ 风险标记（如适用）

**安全原则：**
- ❌ 不提取：密钥、Token、密码、完整连接串、完整用户敏感数据
- ✅ 可以提取：函数签名、模块结构、业务逻辑、API 路径、配置键名

#### CSS 代码面板样式

```css
/* === Code Panel === */
.code-panel {
  border: 1px solid var(--border);
  border-radius: 8px;
  margin: 20px 0;
  overflow: hidden;
}
.code-panel .code-header {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  background: var(--bg-secondary);
  border-bottom: 1px solid var(--border);
  font-size: 13px;
}
.code-panel .code-header .file-icon { margin-right: 6px; }
.code-panel .code-header .file-path {
  flex: 1;
  font-family: 'SF Mono', 'Fira Code', monospace;
  color: var(--text-secondary);
}
.code-panel .code-header .copy-btn {
  padding: 2px 10px;
  font-size: 12px;
  background: var(--bg-primary);
  border: 1px solid var(--border);
  border-radius: 4px;
  cursor: pointer;
  color: var(--text-secondary);
}
.code-panel .code-header .copy-btn:hover {
  background: var(--accent);
  color: #fff;
}
.code-panel pre.code-content {
  margin: 0;
  padding: 16px;
  background: var(--code-bg);
  overflow-x: auto;
  font-size: 13px;
  line-height: 1.6;
  counter-reset: line;
}
.code-panel pre.code-content code {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
}
.code-panel .code-analysis {
  padding: 16px;
  background: var(--analysis-bg);
  border-top: 2px solid var(--accent);
}
.code-panel .code-analysis .analysis-title {
  font-weight: 600;
  margin-bottom: 8px;
  font-size: 14px;
}
.code-panel .code-analysis .analysis-body {
  font-size: 14px;
  line-height: 1.7;
  color: var(--text-primary);
}
.code-panel .code-analysis .note {
  margin-top: 8px;
  padding: 8px 12px;
  background: #fef3c7;
  border-left: 3px solid var(--warning);
  border-radius: 0 4px 4px 0;
  font-size: 13px;
}
[data-theme="dark"] .code-panel .code-analysis .note {
  background: #422006;
  border-left-color: #f59e0b;
  color: #fcd34d;
}
```

### 6. 核心模块详解
- 每个模块一个卡片
- 包含：职责、关键文件、API 列表、依赖关系
- 折叠式详情

### 6. 数据流追踪
- Mermaid 时序图
- 关键步骤代码位置链接

### 7. 埋点事件清单
- 可搜索、可过滤的表格
- 按类别分组（页面浏览 / 用户行为 / 错误事件）
- 标注已确认和待确认

### 8. 开发工作流
- 分支策略
- 代码规范
- 提交规范
- Code Review 流程

### 9. 新人阅读路线
- 按角色推荐的阅读顺序
  - 前端开发者路线
  - 后端开发者路线
  - 全栈开发者路线
- 每步包含：读什么文件 + 做什么练习

### 10. 风险和注意事项
- 风险清单（按严重程度排序）
- 每个风险包含：描述、影响、缓解建议

### 11. 附录
- 常用命令速查
- 环境变量清单
- 外部依赖清单
- 名词解释表

## CSS 设计规范

### 配色方案

```css
/* 亮色主题 */
:root, [data-theme="light"] {
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --bg-sidebar: #1e293b;
  --text-primary: #1e293b;
  --text-secondary: #64748b;
  --text-sidebar: #cbd5e1;
  --border: #e2e8f0;
  --accent: #3b82f6;
  --accent-hover: #2563eb;
  --success: #10b981;
  --warning: #f59e0b;
  --danger: #ef4444;
  --code-bg: #f1f5f9;
}

/* 暗色主题 */
[data-theme="dark"] {
  --bg-primary: #0f172a;
  --bg-secondary: #1e293b;
  --bg-sidebar: #0c1222;
  --text-primary: #e2e8f0;
  --text-secondary: #94a3b8;
  --text-sidebar: #94a3b8;
  --border: #334155;
  --accent: #60a5fa;
  --accent-hover: #93bbfd;
  --code-bg: #1e293b;
}
```

### 布局规范

- 侧边栏宽度：280px（桌面）/ 100%（移动）
- 内容区最大宽度：960px
- 字体：系统默认（`-apple-system, ...`）
- 代码块：等宽字体，带语法高亮行号

### 响应式断点

- 桌面：> 1024px（完整布局）
- 平板：768px - 1024px（侧边栏可折叠）
- 手机：< 768px（侧边栏隐藏，汉堡菜单）

## 交互功能

### 搜索与过滤
- 实时全文搜索（标题、模块名、文件名、函数名）
- 搜索结果高亮
- 支持中文拼音首字母模糊匹配
- 搜索结果分组显示（模块 / 文件 / 埋点）

### 目录树
- 点击展开/折叠子目录
- 全部展开 / 全部折叠按钮
- 当前章节自动展开并高亮
- 滚动时导航跟随高亮（Intersection Observer）

### 代码块
- 一键复制按钮
- 语法高亮（基本关键字高亮）
- 文件路径标注在代码块顶部

### 主题切换
- 点击切换亮色/暗色
- 偏好存储在 localStorage
- 首次访问跟随系统主题

### 快捷键
- `Ctrl/Cmd + K` — 聚焦搜索
- `Ctrl/Cmd + /` — 切换侧边栏
- `Esc` — 关闭搜索/侧边栏（移动端）

## 生成流程

1. 完成项目的只读分析，收集所有章节数据
2. 在目标项目根目录执行 `mkdir -p project-guide`
3. 将分析数据填充到 HTML 模板中：
   - 项目概览数据 → 概览卡片
   - 目录结构分析结果 → 目录树 JSON
   - 模块分析结果 → 模块详情卡片
   - 埋点搜索结果 → 埋点清单表格
   - 架构分析结果 → Mermaid 图表定义
   - 新人路线 → 阅读路线列表
4. 写入 `project-guide/index.html`
5. 告知用户文件路径，建议用浏览器打开

## 模板示例

完整的 HTML 模板参考（在生成时按需调整）：

```html
<!DOCTYPE html>
<html lang="zh-CN" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{PROJECT_NAME} - 项目指南</title>
  <script>
    // Mermaid 初始化（在 head 中提前执行）
    // 实际生成时使用 CDN 加载
  </script>
  <style>
    /* === Reset & Base === */
    *,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
    body{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;background:var(--bg-primary);color:var(--text-primary);display:flex;min-height:100vh}
    :root,[data-theme="light"]{--bg-primary:#fff;--bg-secondary:#f8f9fa;--bg-sidebar:#1e293b;--text-primary:#1e293b;--text-secondary:#64748b;--text-sidebar:#cbd5e1;--border:#e2e8f0;--accent:#3b82f6;--accent-hover:#2563eb;--success:#10b981;--warning:#f59e0b;--danger:#ef4444;--code-bg:#f1f5f9}
    [data-theme="dark"]{--bg-primary:#0f172a;--bg-secondary:#1e293b;--bg-sidebar:#0c1222;--text-primary:#e2e8f0;--text-secondary:#94a3b8;--text-sidebar:#94a3b8;--border:#334155;--accent:#60a5fa;--accent-hover:#93bbfd;--code-bg:#1e293b}
    /* === Top Bar === */
    .top-bar{position:fixed;top:0;left:0;right:0;height:56px;background:var(--bg-primary);border-bottom:1px solid var(--border);display:flex;align-items:center;padding:0 16px;z-index:100;gap:12px}
    .top-bar h1{font-size:18px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
    .menu-toggle{display:none;background:none;border:none;font-size:22px;cursor:pointer;color:var(--text-primary);padding:4px}
    .header-tools{display:flex;align-items:center;gap:8px;margin-left:auto;flex-shrink:0}
    .search-input{padding:6px 12px;border:1px solid var(--border);border-radius:6px;font-size:14px;width:220px;background:var(--bg-secondary);color:var(--text-primary);outline:none;transition:border-color .2s}
    .search-input:focus{border-color:var(--accent)}
    .theme-toggle{background:none;border:none;font-size:20px;cursor:pointer;padding:4px}
    /* === Sidebar === */
    .sidebar{position:fixed;top:56px;left:0;bottom:0;width:280px;background:var(--bg-sidebar);color:var(--text-sidebar);overflow-y:auto;z-index:90;transition:transform .3s}
    .nav-section{padding:16px}
    .nav-title{font-size:12px;font-weight:600;text-transform:uppercase;letter-spacing:.5px;color:#64748b;margin-bottom:8px}
    .nav-item{display:block;padding:6px 12px;color:var(--text-sidebar);text-decoration:none;font-size:14px;border-radius:4px;cursor:pointer;transition:background .15s}
    .nav-item:hover{background:rgba(255,255,255,.1);color:#fff}
    .nav-item.active{background:var(--accent);color:#fff}
    .tree-item{padding-left:0}
    .tree-children{padding-left:16px}
    .tree-toggle{cursor:pointer;user-select:none;font-size:12px;margin-right:4px}
    /* === Main Content === */
    .content{margin-left:280px;margin-top:56px;padding:32px 40px;max-width:960px;width:100%}
    .content section{margin-bottom:40px}
    .content h2{font-size:24px;margin-bottom:16px;padding-bottom:8px;border-bottom:2px solid var(--border)}
    .content h3{font-size:18px;margin:20px 0 12px}
    .content h4{font-size:16px;margin:16px 0 8px}
    /* === Cards === */
    .card{border:1px solid var(--border);border-radius:8px;padding:20px;margin-bottom:16px;background:var(--bg-secondary)}
    .card-title{font-size:16px;font-weight:600;margin-bottom:8px}
    /* === Tags === */
    .tag{display:inline-block;padding:2px 10px;border-radius:12px;font-size:12px;font-weight:500;margin:2px}
    .tag-blue{background:#dbeafe;color:#1e40af}
    .tag-green{background:#d1fae5;color:#065f46}
    .tag-yellow{background:#fef3c7;color:#92400e}
    .tag-red{background:#fee2e2;color:#991b1b}
    [data-theme="dark"] .tag-blue{background:#1e3a5f;color:#93bbfd}
    [data-theme="dark"] .tag-green{background:#064e3b;color:#6ee7b7}
    [data-theme="dark"] .tag-yellow{background:#78350f;color:#fcd34d}
    [data-theme="dark"] .tag-red{background:#7f1d1d;color:#fca5a5}
    /* === Table === */
    table{width:100%;border-collapse:collapse;font-size:14px}
    th,td{padding:8px 12px;text-align:left;border:1px solid var(--border)}
    th{background:var(--bg-secondary);font-weight:600}
    /* === Code Block === */
    .code-block{position:relative;background:var(--code-bg);border:1px solid var(--border);border-radius:6px;margin:12px 0}
    .code-block .file-path{padding:6px 12px;font-size:12px;color:var(--text-secondary);border-bottom:1px solid var(--border);font-family:monospace}
    .code-block pre{padding:12px;overflow-x:auto;font-size:13px;line-height:1.5}
    .code-block .copy-btn{position:absolute;top:4px;right:4px;padding:2px 8px;font-size:12px;background:var(--bg-secondary);border:1px solid var(--border);border-radius:4px;cursor:pointer;opacity:0;transition:opacity .2s}
    .code-block:hover .copy-btn{opacity:1}
    /* === Overlay === */
    .overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,.5);z-index:89}
    /* === Responsive === */
    @media(max-width:1024px){.sidebar{transform:translateX(-100%)}.sidebar.open{transform:translateX(0)}.content{margin-left:0;padding:24px 16px}.menu-toggle{display:block}.overlay.active{display:block}}
    /* === Print === */
    @media print{.sidebar,.top-bar{display:none}.content{margin-left:0;margin-top:0}}
  </style>
</head>
<body>
  <!--
    ============================================
    此处由 skill 分析结果动态填充内容
    包含：导航、概览、结构树、模块、埋点、路线等
    ============================================
  -->
  <script>
    // === 交互逻辑 ===
    // 侧边栏 / 搜索 / 主题切换 / 代码复制 / 目录树操作
    // 由 skill 按项目实际分析结果生成对应的交互逻辑
  </script>
</body>
</html>
```

## 注意事项

- HTML 文件不宜超过 500KB，如果项目很大应精简内容或分页
- Mermaid 图表不宜过多（建议 ≤5 张），避免影响页面加载
- 确保代码块中的敏感信息（密钥、Token）已被移除
- 建议用户在浏览器中打开，而不是 VS Code 内置预览
- 生成日期和 skill 版本标注在页面底部
