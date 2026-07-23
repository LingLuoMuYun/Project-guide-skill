# 模块化 HTML 项目指南架构

深层分析（选项 2）生成的不再是单一 `index.html`，而是一个结构化的 **`project-guide/` 目录**。核心采用 **Section 注册表模式**：每个分析章节是独立文件，通过统一接口注册到核心框架。新增分析维度只需添加 section 文件 + 数据字段 + 一行 `<script>`，无需改动核心。

## 目录结构

深层分析在目标项目根目录生成以下文件夹：

```text
{目标项目根目录}/
└── project-guide/
    ├── index.html                  # 入口文件（薄壳，~50 行）
    ├── css/
    │   └── guide.css               # 所有样式（按节分区，含完整注释）
    ├── js/
    │   ├── guide-core.js           # 核心框架：Section 注册表 + UI 工具
    │   ├── guide-data.js           # 项目分析数据（单一数据源）
    │   └── sections/               # Section 渲染器（可独立增删）
    │       ├── overview.js         # 1. 项目概览
    │       ├── quickstart.js       # 2. 快速开始
    │       ├── structure.js        # 3. 项目结构树
	    │       ├── concept-map.js      # 4. 概念地图 🧭（业务概念→代码位置）
    │       ├── architecture.js     # 5. 架构概览
    │       ├── code-analysis.js    # 6. 代码展示与分析 ⭐（深层核心）
    │       ├── modules.js          # 7. 核心模块详解
    │       ├── dataflow.js         # 8. 数据流追踪
    │       ├── tracking.js         # 9. 埋点事件清单
    │       ├── workflow.js         # 10. 开发工作流
    │       ├── roadmap.js          # 11. 新人阅读路线
    │       ├── risks.js            # 12. 风险和注意事项
    │       └── appendix.js         # 13. 附录
    └── extensions/
        └── README.md               # 扩展开发指南
```

**兼容性保证：**
- 所有 JS 通过 `<script src="...">` 加载，兼容 `file://` 协议（无需本地服务器）
- Mermaid 通过 CDN 加载（可选；无网络时图表区域降级为静态文本）
- 所有交互功能与原单文件版本一致：搜索/主题切换/目录树/快捷键/响应式

---

## 1. 核心框架模板：`js/guide-core.js`

这是整个指南的运行引擎。提供 Section 注册表、生命周期管理和 UI 工具集。

```javascript
/**
 * ProjectGuide — 模块化项目指南核心框架
 * 
 * 职责：
 * - 管理 section 注册和渲染
 * - 提供 UI 工具（侧边栏、搜索、主题、目录树、快捷键）
 * - 协调页面生命周期
 *
 * 扩展方式：
 *   调用 ProjectGuide.registerSection({ id, title, icon, priority, render })
 *   详见 extensions/README.md
 */
;(function() {
  'use strict';

  // ============================================================
  // Section 注册表
  // ============================================================
  var sections = [];        // 按 priority 排序后的 section 列表
  var sectionsById = {};    // id → section 快速查找
  var mainContent = null;
  var sidebarNav = null;

  /**
   * 注册一个 section。
   *
   * @param {Object} config
   * @param {string}   config.id        - 唯一标识（用作 DOM id 和锚点）
   * @param {string}   config.title     - 导航中显示的名称
   * @param {string}   config.icon      - 图标 emoji
   * @param {number}   [config.priority=99] - 排序权重（越小越靠前）
   * @param {Function} config.render    - 渲染函数 (containerElement, projectData) => void
   * @param {Function} [config.onNavClick] - 可选：点击导航时的自定义行为
   */
  function registerSection(config) {
    if (!config.id || !config.title || !config.render) {
      console.error('[ProjectGuide] registerSection 缺少必填字段 (id/title/render):', config);
      return;
    }
    if (sectionsById[config.id]) {
      console.warn('[ProjectGuide] section "' + config.id + '" 被重复注册，后者覆盖前者');
    }
    config.priority = typeof config.priority === 'number' ? config.priority : 99;
    sectionsById[config.id] = config;
  }

  // ============================================================
  // 生命周期
  // ============================================================

  /**
   * 初始化整个指南页面。
   * 必须在所有 section 文件和数据文件加载完成后调用。
   */
  function init() {
    mainContent = document.getElementById('mainContent');
    sidebarNav  = document.getElementById('sidebarNav');

    // 按 priority 排序
    sections = Object.values(sectionsById).sort(function(a, b) {
      return a.priority - b.priority;
    });

    buildSidebar();
    renderAllSections();
    initUI();
  }

  /**
   * 从已注册 section 构建侧边栏导航
   */
  function buildSidebar() {
    if (!sidebarNav) return;
    var html = '';
    sections.forEach(function(sec) {
      html += '<a class="nav-item" href="#section-' + sec.id + '" data-section="' + sec.id + '">' +
              '<span class="nav-icon">' + (sec.icon || '📄') + '</span>' +
              sec.title +
              '</a>';
    });
    sidebarNav.innerHTML = html;
  }

  /**
   * 渲染所有已注册 section
   */
  function renderAllSections() {
    if (!mainContent) return;
    var data = window.__PROJECT_DATA__ || {};

    sections.forEach(function(sec) {
      // 创建 section 容器
      var sectionEl = document.createElement('section');
      sectionEl.id = 'section-' + sec.id;
      sectionEl.setAttribute('data-section', sec.id);

      // 标题
      var titleEl = document.createElement('h2');
      titleEl.innerHTML = (sec.icon || '') + ' ' + sec.title;
      sectionEl.appendChild(titleEl);

      // 内容容器（传给 render 函数）
      var bodyEl = document.createElement('div');
      bodyEl.className = 'section-body';
      sectionEl.appendChild(bodyEl);

      mainContent.appendChild(sectionEl);

      // 调用 section 的渲染函数
      try {
        sec.render(bodyEl, data);
      } catch (e) {
        bodyEl.innerHTML = '<p style="color:var(--danger)">⚠ 渲染出错：' + e.message + '</p>';
        console.error('[ProjectGuide] section "' + sec.id + '" 渲染失败:', e);
      }
    });
  }

  // ============================================================
  // UI 工具集
  // ============================================================

  function initUI() {
    initSidebarToggle();
    initSearch();
    initTheme();
    initTreeInteraction();
    initKeyboardShortcuts();
    initScrollSpy();
  }

  // --- 侧边栏切换 ---
  function initSidebarToggle() {
    var toggle = document.getElementById('menuToggle');
    var sidebar = document.getElementById('sidebar');
    var overlay = document.getElementById('overlay');
    if (!toggle || !sidebar) return;

    toggle.addEventListener('click', function() {
      sidebar.classList.toggle('open');
      if (overlay) overlay.classList.toggle('active');
    });
    if (overlay) {
      overlay.addEventListener('click', function() {
        sidebar.classList.remove('open');
        overlay.classList.remove('active');
      });
    }
  }

  // --- 全文搜索 ---
  function initSearch() {
    var input = document.getElementById('searchInput');
    if (!input) return;

    input.addEventListener('input', debounce(function() {
      var query = this.value.toLowerCase().trim();
      var allSections = document.querySelectorAll('.content > section');
      var navItems = document.querySelectorAll('.nav-item');

      if (!query) {
        // 恢复全部
        allSections.forEach(function(s) { s.style.display = ''; });
        navItems.forEach(function(n) { n.style.display = ''; });
        return;
      }

      var matchedIds = {};
      allSections.forEach(function(section) {
        var text = section.textContent.toLowerCase();
        var id = section.getAttribute('data-section');
        if (text.indexOf(query) !== -1) {
          section.style.display = '';
          if (id) matchedIds[id] = true;
        } else {
          section.style.display = 'none';
        }
      });

      navItems.forEach(function(nav) {
        var sid = nav.getAttribute('data-section');
        nav.style.display = (sid && matchedIds[sid]) ? '' : 'none';
      });
    }, 200));
  }

  // --- 主题切换 ---
  function initTheme() {
    var toggle = document.getElementById('themeToggle');
    if (!toggle) return;
    var html = document.documentElement;

    // 读取偏好
    var saved = localStorage.getItem('project-guide-theme');
    if (saved === 'dark') {
      html.setAttribute('data-theme', 'dark');
    } else if (!saved) {
      // 跟随系统
      if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
        html.setAttribute('data-theme', 'dark');
      }
    }

    toggle.addEventListener('click', function() {
      var current = html.getAttribute('data-theme');
      var next = (current === 'dark') ? 'light' : 'dark';
      html.setAttribute('data-theme', next);
      localStorage.setItem('project-guide-theme', next);
    });
  }

  // --- 目录树交互 ---
  function initTreeInteraction() {
    document.addEventListener('click', function(e) {
      var toggle = e.target.closest('.tree-toggle');
      if (!toggle) return;
      var children = toggle.parentElement.querySelector('.tree-children');
      if (children) {
        var isOpen = children.style.display !== 'none';
        children.style.display = isOpen ? 'none' : 'block';
        toggle.textContent = isOpen ? '▶' : '▼';
      }
    });

    // 全部展开/折叠按钮
    var expandAll = document.getElementById('expandAll');
    var collapseAll = document.getElementById('collapseAll');
    if (expandAll) {
      expandAll.addEventListener('click', function() {
        document.querySelectorAll('.tree-children').forEach(function(c) { c.style.display = 'block'; });
        document.querySelectorAll('.tree-toggle').forEach(function(t) { t.textContent = '▼'; });
      });
    }
    if (collapseAll) {
      collapseAll.addEventListener('click', function() {
        document.querySelectorAll('.tree-children').forEach(function(c) { c.style.display = 'none'; });
        document.querySelectorAll('.tree-toggle').forEach(function(t) { t.textContent = '▶'; });
      });
    }
  }

  // --- 键盘快捷键 ---
  function initKeyboardShortcuts() {
    document.addEventListener('keydown', function(e) {
      // Ctrl/Cmd + K → 搜索
      if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
        e.preventDefault();
        var search = document.getElementById('searchInput');
        if (search) search.focus();
      }
      // Ctrl/Cmd + / → 侧边栏
      if ((e.ctrlKey || e.metaKey) && e.key === '/') {
        e.preventDefault();
        var sidebar = document.getElementById('sidebar');
        var overlay = document.getElementById('overlay');
        if (sidebar) {
          sidebar.classList.toggle('open');
          if (overlay) overlay.classList.toggle('active');
        }
      }
      // Esc → 关闭搜索或侧边栏
      if (e.key === 'Escape') {
        var search = document.getElementById('searchInput');
        var sidebar = document.getElementById('sidebar');
        var overlay = document.getElementById('overlay');
        if (document.activeElement === search) {
          search.blur();
          search.value = '';
          search.dispatchEvent(new Event('input'));
        } else if (sidebar && sidebar.classList.contains('open')) {
          sidebar.classList.remove('open');
          if (overlay) overlay.classList.remove('active');
        }
      }
    });
  }

  // --- 滚动跟随 ---
  function initScrollSpy() {
    var navItems = document.querySelectorAll('.nav-item');
    if (!navItems.length) return;
    var observer = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) {
          var id = entry.target.getAttribute('data-section');
          navItems.forEach(function(nav) {
            nav.classList.toggle('active', nav.getAttribute('data-section') === id);
          });
        }
      });
    }, { rootMargin: '-80px 0px -70% 0px' });

    document.querySelectorAll('.content > section').forEach(function(s) {
      observer.observe(s);
    });
  }

  // --- 工具函数 ---
  function debounce(fn, delay) {
    var timer;
    return function() {
      var ctx = this, args = arguments;
      clearTimeout(timer);
      timer = setTimeout(function() { fn.apply(ctx, args); }, delay);
    };
  }

  // ============================================================
  // 公开 API
  // ============================================================
  window.ProjectGuide = {
    registerSection: registerSection,
    init: init,
    // 辅助工具（section 渲染时可用）
    helpers: {
      escapeHtml: function(str) {
        return String(str)
          .replace(/&/g, '&amp;')
          .replace(/</g, '&lt;')
          .replace(/>/g, '&gt;')
          .replace(/"/g, '&quot;');
      },
      debounce: debounce
    }
  };

})();
```

---

## 2. 数据文件模板：`js/guide-data.js`

所有项目分析数据集中在这个文件中。每个顶层字段对应一个或多个 section 的数据源。Agent 在生成指南时根据实际分析结果填充。

```javascript
/**
 * 项目分析数据
 * 
 * 所有分析结果集中在此。添加新 section 时在此文件中增加对应字段。
 * ⚠ 生成的代码中不得包含密钥、Token、密码等敏感值。
 */
window.__PROJECT_DATA__ = {
  // ==========================================
  // 元数据（跨会话缓存、增量分析使用）
  // ==========================================
  _meta: {
    /** 分析日期 ISO 字符串 */
    analyzedAt: '{2026-07-23T10:30:00Z}',
    /** 分析时的 git commit hash（增量更新的基线）*/
    gitCommit: '{abc123def456}',
    /** 分析时的文件总数 */
    fileCount: 0,
    /** 分析模式：'shallow' | 'deep' | 'focused' */
    analysisMode: 'deep',
    /** 项目根目录的绝对路径（跨会话匹配用）*/
    projectRoot: '{/absolute/path/to/project}'
  },

  // ==========================================
  // 项目基本信息（overview section 使用）
  // ==========================================
  project: {
    name: '{项目名称}',
    description: '{一句话描述}',
    targetUsers: '{目标用户}',
    maturity: '{当前成熟度，如：生产环境 / 开发中 / 原型阶段}',
    generatedAt: '{生成日期，如 2026-07-23}'
  },

  // ==========================================
  // 技术栈（overview section 使用）
  // ==========================================
  techStack: [
    { category: '语言/运行时', tech: '{技术名称}', evidence: '{证据：文件路径}' },
    { category: '框架', tech: '{技术名称}', evidence: '{证据}' },
    { category: '构建工具', tech: '{技术名称}', evidence: '{证据}' },
    { category: '包管理器', tech: '{技术名称}', evidence: '{证据}' }
    // ...
  ],

  // ==========================================
  // 快速开始（quickstart section 使用）
  // ==========================================
  quickstart: {
    requirements: '{Node.js >= 18, pnpm >= 8, ...}',
    install: 'pnpm install',
    dev: 'pnpm run dev',
    build: 'pnpm run build',
    test: 'pnpm run test',
    urls: [
      { label: '开发环境', url: 'http://localhost:3000' },
      { label: '生产环境', url: 'https://example.com' }
    ]
  },

  // ==========================================
  // 项目结构树（structure section 使用）
  // ==========================================
  structure: {
    /**
     * 树节点定义：
     * { name, icon?, priority?, note?, children? }
     * priority: 'P0' | 'P1' | 'P2' | 'P3'
     */
    tree: [
      {
        name: 'src/',
        icon: '📁',
        priority: 'P0',
        note: '核心源代码',
        children: [
          {
            name: 'main.ts',
            icon: '★',
            priority: 'P0',
            note: '应用入口 — 初始化 Vue 实例和全局插件'
          },
          {
            name: 'router/',
            icon: '📁',
            priority: 'P0',
            note: '路由定义',
            children: [
              { name: 'index.ts', priority: 'P0', note: '完整路由表配置' }
            ]
          }
          // ...
        ]
      }
      // ...
    ],
    dependencyFlow: '页面层 → 组件/Hook层 → 服务/状态层 → 工具/类型层',
    warnings: [
      { type: 'circular', description: '...' },
      { type: 'god-module', description: '...' }
    ]
  },

  // ==========================================
  // 概念地图（concept-map section 使用）🧭
  // ==========================================
  conceptMap: {
    /** Mermaid 概念关系图（LR 方向），节点为业务概念，边为依赖/数据流关系。null 表示无概念地图 */
    mermaidDef: 'graph LR
  Auth[用户认证] -->|授权| Permission[权限控制]
  Auth -->|写入| UserStore[用户状态]
  Order[订单管理] -->|发起| Payment[支付流程]',
    /** 概念详情列表 */
    concepts: [
      {
        id: 'auth',
        name: '用户认证',
        description: '处理用户登录、注册、Token 刷新、SSO 对接',
        priority: 'P0',
        files: [
          { path: 'src/pages/Login/', desc: '登录页面' },
          { path: 'src/services/authService.ts', desc: '认证 API 服务' },
          { path: 'src/store/userStore.ts', desc: '用户状态管理' },
          { path: 'src/hooks/useAuth.ts', desc: '认证 Hook' }
        ],
        relatedConcepts: ['permission', 'user-profile']
      }
      // ... 更多概念
    ]
  },

  // ==========================================
  // 架构图（architecture section 使用）
  // ==========================================
  architecture: {
    /** Mermaid 图表定义（原始文本），null 表示无架构图 */
    mermaidDef: 'graph TB\\n  A[用户] --> B[页面层]\\n  B --> C[服务层]\\n  C --> D[后端API]',
    layers: [
      { name: '页面层', description: '...', files: ['...'] },
      { name: '服务层', description: '...', files: ['...'] }
    ]
  },

  // ==========================================
  // 代码片段与分析（code-analysis section 使用）⭐ 深层核心
  // ==========================================
  codeSnippets: [
    {
      file: 'src/services/userService.ts',
      lines: '45-67',
      code: 'export async function login(params) {\n  const res = await request.post(\n    \'/api/auth/login\', params\n  );\n  if (res.code === 0) {\n    setToken(res.data.token);\n    userStore.setUser(res.data.user);\n  }\n  return res;\n}',
      language: 'typescript',
      analysis: {
        summary: '发送登录请求，成功后存储 token 并更新全局用户状态',
        lineNotes: [
          { lines: 'L1', text: '异步函数声明，接收登录凭据参数' },
          { lines: 'L2-L4', text: '调用封装的 HTTP 实例发送 POST 请求到 /api/auth/login' },
          { lines: 'L5-L8', text: '响应码为 0 表示成功，执行：setToken 写入持久化存储，userStore.setUser 更新全局状态。⚠ 两个操作无事务保证，需关注顺序' },
          { lines: 'L9', text: '返回完整响应给上层调用方' }
        ],
        inputs: '{ username: string, password: string, captcha?: string }',
        outputs: '{ code: number, data: { token: string, user: object }, msg: string }',
        callers: ['LoginPage.vue', 'AuthModal.vue'],
        notes: '未处理网络超时和断网情况，调用方需自行 catch'
      }
    }
    // 更多代码片段...
  ],

  // ==========================================
  // 核心模块（modules section 使用）
  // ==========================================
  modules: [
    {
      name: '{模块名}',
      responsibility: '{一句话职责}',
      priority: 'P0',
      keyFiles: ['src/services/xxx.ts', 'src/pages/Xxx/'],
      apis: ['functionA()', 'functionB()'],
      dependencies: ['src/utils/request.ts'],
      risks: ['{风险描述}']
    }
    // ...
  ],

  // ==========================================
  // 数据流（dataflow section 使用）
  // ==========================================
  dataflow: {
    mermaidDef: 'sequenceDiagram\\n  participant U as 用户\\n  participant P as 页面\\n  participant S as 服务层\\n  participant API as 后端\\n  U->>P: 点击登录\\n  P->>S: login(params)\\n  S->>API: POST /api/auth/login\\n  API-->>S: { token, user }\\n  S->>S: setToken()\\n  S-->>P: 返回结果\\n  P->>P: 跳转首页',
    steps: [
      { order: 1, actor: '用户', action: '点击登录按钮', location: 'LoginPage.vue:45' },
      { order: 2, actor: 'LoginPage', action: '调用 authService.login()', location: 'LoginPage.vue:52' }
      // ...
    ]
  },

  // ==========================================
  // 埋点事件（tracking section 使用）
  // ==========================================
  tracking: {
    sdks: ['{识别的埋点 SDK 名称，如 @sensors/datafunchannel}'],
    events: [
      {
        eventName: 'page_view',
        category: '页面浏览',
        trigger: '路由切换时',
        location: 'src/router/index.ts:45',
        params: '{ page_name: string }',
        status: 'confirmed'
      }
      // ...
    ],
    gaps: [
      { flow: '注册流程', expected: '每步转化率埋点', actual: '仅有成功上报', suggestion: '在步骤1/2/3中增加中间事件' }
    ]
  },

  // ==========================================
  // 开发工作流（workflow section 使用）
  // ==========================================
  workflow: {
    branching: '{分支策略，如 Git Flow / Trunk-based}',
    conventions: {
      naming: '{命名规范}',
      commit: '{提交信息规范，如 Conventional Commits}',
      codeStyle: '{代码风格工具，如 ESLint + Prettier}'
    },
    review: '{Code Review 流程说明}',
    ci: '{CI/CD 简要说明}'
  },

  // ==========================================
  // 新人阅读路线（roadmap section 使用）
  // ==========================================
  roadmap: {
    frontend: [
      { step: 1, action: '阅读 src/router/index.ts', goal: '理解路由结构' },
      { step: 2, action: '阅读 src/services/ 下的核心 service', goal: '理解 API 调用方式' }
      // ...
    ],
    backend: [
      { step: 1, action: '...', goal: '...' }
      // ...
    ],
    fullstack: [
      { step: 1, action: '...', goal: '...' }
      // ...
    ]
  },

  // ==========================================
  // 风险项（risks section 使用）
  // ==========================================
  risks: [
    {
      severity: 'high',
      description: '{风险描述}',
      evidence: '{文件路径或配置中的证据}',
      impact: '{影响范围}',
      suggestion: '{缓解建议}'
    }
    // ...
  ],

  // ==========================================
  // 附录（appendix section 使用）
  // ==========================================
  appendix: {
    commands: [
      { name: 'pnpm dev', description: '启动开发服务器' },
      { name: 'pnpm build', description: '生产构建' }
      // ...
    ],
    envVars: [
      { name: 'VITE_API_BASE', description: 'API 基础地址', defaultValue: 'http://localhost:8080' }
      // ...
    ],
    glossary: [
      { term: '{缩写/术语}', definition: '{全称和解释}' }
      // ...
    ]
  }
};
```

---

## 3. 入口文件模板：`index.html`

精简的 HTML 壳，负责加载 CSS、JS 并启动应用。生成时替换 `{项目名称}`。

```html
<!DOCTYPE html>
<html lang="zh-CN" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{项目名称} - 项目指南</title>
  <link rel="stylesheet" href="css/guide.css">
  <!-- Mermaid CDN（可选；无网络时图表区域降级为静态文本） -->
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <script>
    // Mermaid 初始化
    document.addEventListener('DOMContentLoaded', function() {
      if (typeof mermaid !== 'undefined') {
        mermaid.initialize({ startOnLoad: true, theme: 'default', securityLevel: 'loose' });
      }
    });
  </script>
</head>
<body>
  <!-- ========== 顶部工具栏 ========== -->
  <header class="top-bar">
    <button class="menu-toggle" id="menuToggle" aria-label="切换菜单">☰</button>
    <h1>{项目名称} <span class="title-sub">项目指南</span></h1>
    <div class="header-tools">
      <input type="text" class="search-input" id="searchInput"
             placeholder="搜索模块、文件、函数..." aria-label="搜索">
      <button class="theme-toggle" id="themeToggle" aria-label="切换主题">🌓</button>
    </div>
  </header>

  <!-- ========== 左侧导航 ========== -->
  <nav class="sidebar" id="sidebar">
    <div class="nav-section">
      <div class="nav-title">📋 快速导航</div>
      <div id="sidebarNav"></div>
    </div>
  </nav>

  <!-- ========== 遮罩层（移动端） ========== -->
  <div class="overlay" id="overlay"></div>

  <!-- ========== 主内容区 ========== -->
  <main class="content" id="mainContent"></main>

  <!-- ========== 页脚 ========== -->
  <footer class="page-footer">
    <p>由 Project Guide Skill 生成 · {生成日期}</p>
  </footer>

  <!-- ============================================================
       脚本加载顺序：
       1. 核心框架 (guide-core.js)
       2. 项目数据 (guide-data.js)
       3. Section 渲染器 (js/sections/*.js) — 顺序无关，按 priority 排序
       4. 启动
       ============================================================ -->
  <script src="js/guide-core.js"></script>
  <script src="js/guide-data.js"></script>

  <!-- Section 渲染器 — 可自由增删，无需改动其他文件 -->
  <script src="js/sections/overview.js"></script>
  <script src="js/sections/quickstart.js"></script>
  <script src="js/sections/structure.js"></script>
  <script src="js/sections/concept-map.js"></script>
  <script src="js/sections/architecture.js"></script>
  <script src="js/sections/code-analysis.js"></script>
  <script src="js/sections/modules.js"></script>
  <script src="js/sections/dataflow.js"></script>
  <script src="js/sections/tracking.js"></script>
  <script src="js/sections/workflow.js"></script>
  <script src="js/sections/roadmap.js"></script>
  <script src="js/sections/risks.js"></script>
  <script src="js/sections/appendix.js"></script>

  <!-- 启动 -->
  <script>ProjectGuide.init();</script>
</body>
</html>
```

---

## 4. 样式模板：`css/guide.css`

```css
/**
 * Project Guide — 完整样式表
 *
 * 分区：
 *   1. Reset & CSS Variables (themes)
 *   2. Layout (top bar, sidebar, main content, footer)
 *   3. Navigation (sidebar items, tree)
 *   4. Cards & Tags
 *   5. Code Panels ⭐
 *   6. Tables
 *   7. Mermaid Diagrams
 *   8. Search Results
 *   9. Responsive
 *   10. Concept Map
 *   11. Print
 *   10. Print
 */

/* ============================================================
   1. Reset & CSS Variables
   ============================================================ */
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html{scroll-behavior:smooth}
body{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,"Noto Sans SC",sans-serif;background:var(--bg-primary);color:var(--text-primary);line-height:1.6}

/* 亮色主题 */
:root,[data-theme="light"]{
  --bg-primary:#ffffff;
  --bg-secondary:#f8f9fa;
  --bg-sidebar:#1e293b;
  --text-primary:#1e293b;
  --text-secondary:#64748b;
  --text-sidebar:#cbd5e1;
  --text-sidebar-active:#ffffff;
  --border:#e2e8f0;
  --accent:#3b82f6;
  --accent-hover:#2563eb;
  --accent-light:#eff6ff;
  --success:#10b981;
  --warning:#f59e0b;
  --danger:#ef4444;
  --code-bg:#f1f5f9;
  --analysis-bg:#f0f9ff;
  --analysis-border:#bae6fd;
}

/* 暗色主题 */
[data-theme="dark"]{
  --bg-primary:#0f172a;
  --bg-secondary:#1e293b;
  --bg-sidebar:#0c1222;
  --text-primary:#e2e8f0;
  --text-secondary:#94a3b8;
  --text-sidebar:#94a3b8;
  --text-sidebar-active:#ffffff;
  --border:#334155;
  --accent:#60a5fa;
  --accent-hover:#93bbfd;
  --accent-light:#0c1a2e;
  --code-bg:#1e293b;
  --analysis-bg:#0c1a2e;
  --analysis-border:#1e3a5f;
}
/* 暗色下调整 tag / note 颜色 */
[data-theme="dark"] .tag-blue{background:#1e3a5f;color:#93bbfd}
[data-theme="dark"] .tag-green{background:#064e3b;color:#6ee7b7}
[data-theme="dark"] .tag-yellow{background:#78350f;color:#fcd34d}
[data-theme="dark"] .tag-red{background:#7f1d1d;color:#fca5a5}
[data-theme="dark"] .note{background:#422006;border-left-color:#f59e0b;color:#fcd34d}

/* ============================================================
   2. Layout
   ============================================================ */

/* --- Top Bar --- */
.top-bar{
  position:fixed;top:0;left:0;right:0;height:56px;
  background:var(--bg-primary);border-bottom:1px solid var(--border);
  display:flex;align-items:center;padding:0 16px;z-index:100;gap:12px;
}
.top-bar h1{font-size:18px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.top-bar .title-sub{font-weight:400;color:var(--text-secondary);font-size:14px;margin-left:4px}
.menu-toggle{display:none;background:none;border:none;font-size:22px;cursor:pointer;color:var(--text-primary);padding:4px}
.header-tools{display:flex;align-items:center;gap:8px;margin-left:auto;flex-shrink:0}
.search-input{
  padding:6px 12px;border:1px solid var(--border);border-radius:6px;font-size:14px;
  width:220px;background:var(--bg-secondary);color:var(--text-primary);
  outline:none;transition:border-color .2s;
}
.search-input:focus{border-color:var(--accent)}
.theme-toggle{background:none;border:none;font-size:20px;cursor:pointer;padding:4px}

/* --- Sidebar --- */
.sidebar{
  position:fixed;top:56px;left:0;bottom:0;width:280px;
  background:var(--bg-sidebar);color:var(--text-sidebar);
  overflow-y:auto;z-index:90;transition:transform .3s;
}
.nav-section{padding:16px}
.nav-title{
  font-size:12px;font-weight:600;text-transform:uppercase;letter-spacing:.5px;
  color:#64748b;margin-bottom:8px;
}
.nav-item{
  display:flex;align-items:center;gap:8px;
  padding:8px 12px;color:var(--text-sidebar);text-decoration:none;
  font-size:14px;border-radius:4px;cursor:pointer;transition:background .15s;
  margin-bottom:2px;
}
.nav-item:hover{background:rgba(255,255,255,.1);color:#fff}
.nav-item.active{background:var(--accent);color:var(--text-sidebar-active)}
.nav-icon{font-size:14px;width:20px;text-align:center;flex-shrink:0}

/* --- Main Content --- */
.content{
  margin-left:280px;margin-top:56px;padding:32px 40px;
  max-width:960px;width:100%;min-height:calc(100vh - 56px);
}
.content section{margin-bottom:48px;scroll-margin-top:72px}
.content h2{
  font-size:24px;margin-bottom:16px;padding-bottom:8px;
  border-bottom:2px solid var(--border);display:flex;align-items:center;gap:8px;
}
.content h3{font-size:18px;margin:24px 0 12px;color:var(--text-primary)}
.content h4{font-size:16px;margin:16px 0 8px;color:var(--text-secondary)}

/* --- Footer --- */
.page-footer{
  margin-left:280px;padding:16px 40px 32px;max-width:960px;
  text-align:center;font-size:13px;color:var(--text-secondary);
  border-top:1px solid var(--border);
}

/* --- Overlay --- */
.overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,.5);z-index:89}

/* ============================================================
   3. Navigation — Tree
   ============================================================ */
.tree-container{font-family:'SF Mono','Fira Code','Cascadia Code',monospace;font-size:13px;line-height:1.8}
.tree-item{padding:2px 0}
.tree-toggle{cursor:pointer;user-select:none;font-size:12px;margin-right:4px;display:inline-block;width:16px;text-align:center}
.tree-children{padding-left:20px}
.tree-label{display:inline}
.tree-label .tree-name{font-weight:500}
.tree-note{color:var(--text-secondary);font-size:12px;margin-left:8px}
.tree-badge{display:inline-block;padding:0 6px;border-radius:3px;font-size:10px;font-weight:600;margin-left:4px}
.tree-badge.p0{background:#dbeafe;color:#1e40af}
.tree-badge.p1{background:#d1fae5;color:#065f46}
.tree-badge.p2{background:#fef3c7;color:#92400e}
.tree-badge.p3{background:#f1f5f9;color:#64748b}
[data-theme="dark"] .tree-badge.p0{background:#1e3a5f;color:#93bbfd}
[data-theme="dark"] .tree-badge.p1{background:#064e3b;color:#6ee7b7}
[data-theme="dark"] .tree-badge.p2{background:#78350f;color:#fcd34d}
[data-theme="dark"] .tree-badge.p3{background:#334155;color:#94a3b8}

.tree-controls{margin-bottom:10px;display:flex;gap:8px}
.tree-controls button{
  padding:4px 12px;font-size:12px;background:var(--bg-secondary);
  border:1px solid var(--border);border-radius:4px;cursor:pointer;
  color:var(--text-secondary);
}
.tree-controls button:hover{background:var(--accent);color:#fff;border-color:var(--accent)}

/* ============================================================
   4. Cards & Tags
   ============================================================ */
.card{
  border:1px solid var(--border);border-radius:8px;padding:20px;
  margin-bottom:16px;background:var(--bg-secondary);
}
.card-title{font-size:16px;font-weight:600;margin-bottom:8px}

/* Tags */
.tag{display:inline-block;padding:2px 10px;border-radius:12px;font-size:12px;font-weight:500;margin:2px}
.tag-blue{background:#dbeafe;color:#1e40af}
.tag-green{background:#d1fae5;color:#065f46}
.tag-yellow{background:#fef3c7;color:#92400e}
.tag-red{background:#fee2e2;color:#991b1b}
.tag-gray{background:#f1f5f9;color:#475569}

/* Tag cloud */
.tag-cloud{display:flex;flex-wrap:wrap;gap:6px;margin:12px 0}

/* ============================================================
   5. Code Panels ⭐（深层分析核心组件）
   ============================================================ */
.code-panel{
  border:1px solid var(--border);border-radius:8px;
  margin:20px 0;overflow:hidden;
}
.code-panel + .code-panel{margin-top:24px}

/* 代码头部（文件路径 + 复制按钮） */
.code-panel .code-header{
  display:flex;align-items:center;padding:8px 12px;
  background:var(--bg-secondary);border-bottom:1px solid var(--border);
  font-size:13px;
}
.code-panel .code-header .file-icon{margin-right:6px}
.code-panel .code-header .file-path{
  flex:1;font-family:'SF Mono','Fira Code',monospace;
  color:var(--text-secondary);font-size:12px;
}
.code-panel .code-header .copy-btn{
  padding:2px 10px;font-size:12px;background:var(--bg-primary);
  border:1px solid var(--border);border-radius:4px;cursor:pointer;
  color:var(--text-secondary);transition:all .15s;
}
.code-panel .code-header .copy-btn:hover{background:var(--accent);color:#fff;border-color:var(--accent)}

/* 代码内容区 */
.code-panel .code-content{
  margin:0;padding:16px;background:var(--code-bg);
  overflow-x:auto;font-size:13px;line-height:1.7;
}
.code-panel .code-content code{
  font-family:'SF Mono','Fira Code','Cascadia Code','JetBrains Mono',monospace;
  white-space:pre;
}

/* 分析注释区 */
.code-panel .code-analysis{
  padding:16px;background:var(--analysis-bg);
  border-top:2px solid var(--analysis-border);
}
.code-panel .code-analysis .analysis-title{
  font-weight:600;margin-bottom:10px;font-size:14px;
}
.code-panel .code-analysis .analysis-body{
  font-size:14px;line-height:1.7;color:var(--text-primary);
}
.code-panel .code-analysis .analysis-body ul{
  padding-left:20px;margin:8px 0;
}
.code-panel .code-analysis .analysis-body li{
  margin-bottom:4px;
}
.code-panel .code-analysis .analysis-body li strong{
  color:var(--accent);
}
.code-panel .code-analysis .analysis-field{
  margin-bottom:6px;
}
.code-panel .code-analysis .analysis-field strong{
  display:inline-block;min-width:70px;color:var(--text-secondary);
}
/* 注意事项块 */
.code-panel .code-analysis .note{
  margin-top:10px;padding:10px 14px;
  background:#fef3c7;border-left:3px solid var(--warning);
  border-radius:0 6px 6px 0;font-size:13px;
}
.code-panel .code-analysis .note::before{content:'⚠ '}

/* ============================================================
   6. Tables
   ============================================================ */
table{width:100%;border-collapse:collapse;font-size:14px;margin:12px 0}
th,td{padding:10px 14px;text-align:left;border:1px solid var(--border)}
th{background:var(--bg-secondary);font-weight:600;white-space:nowrap}
tr:hover{background:var(--accent-light)}
/* 可搜索表格 */
.table-search{margin-bottom:12px}
.table-search input{
  padding:6px 12px;border:1px solid var(--border);border-radius:6px;
  font-size:14px;width:280px;background:var(--bg-secondary);color:var(--text-primary);
  outline:none;
}
.table-search input:focus{border-color:var(--accent)}

/* ============================================================
   7. Mermaid Diagrams
   ============================================================ */
.mermaid-wrapper{
  margin:20px 0;padding:20px;background:var(--bg-secondary);
  border:1px solid var(--border);border-radius:8px;overflow-x:auto;
}
.mermaid-wrapper .mermaid{display:flex;justify-content:center}

/* ============================================================
   8. Search
   ============================================================ */
.search-highlight{background:#fde68a;padding:1px 2px;border-radius:2px}
[data-theme="dark"] .search-highlight{background:#854d0e;color:#fef3c7}

/* ============================================================
   9. Responsive
   ============================================================ */
@media(max-width:1024px){
  .sidebar{transform:translateX(-100%)}
  .sidebar.open{transform:translateX(0)}
  .content{margin-left:0;padding:24px 20px}
  .page-footer{margin-left:0;padding:16px 20px 32px}
  .menu-toggle{display:block}
  .overlay.active{display:block}
  .search-input{width:160px}
}

@media(max-width:640px){
  .top-bar h1{font-size:15px}
  .top-bar .title-sub{display:none}
  .search-input{width:120px}
  .content{padding:16px 12px}
  .content h2{font-size:20px}
}

/* ============================================================
   10. Concept Map
   ============================================================ */
.concept-grid{
  display:grid;grid-template-columns:repeat(2, 1fr);gap:16px;margin:16px 0;
}
.concept-card{
  scroll-margin-top:80px;transition:box-shadow .3s;
}
.concept-card .file-link{
  color:var(--accent);text-decoration:none;
}
.concept-card .file-link:hover{text-decoration:underline}
.concept-card .file-link code{
  font-family:'SF Mono','Fira Code',monospace;font-size:12px;
  background:var(--code-bg);padding:1px 6px;border-radius:3px;
}
.related-tag{
  display:inline-block;padding:2px 10px;border-radius:12px;font-size:12px;
  background:var(--accent-light);color:var(--accent);
  text-decoration:none;margin:1px 2px;transition:background .15s;
}
.related-tag:hover{background:var(--accent);color:#fff}
/* Mermaid 概念图节点可点击 */
.concept-grid .mermaid-wrapper + * {margin-top:20px}
@media(max-width:768px){
  .concept-grid{grid-template-columns:1fr}
}

/* ============================================================
   11. Print
   ============================================================ */
@media print{
  .sidebar,.top-bar,.overlay,.page-footer{display:none}
  .content{margin-left:0;margin-top:0;padding:0;max-width:none}
  .code-panel{border:1px solid #ccc;break-inside:avoid}
  .code-panel .code-analysis{background:#fff}
}
```

---

## 5. Section 渲染器模板

每个 section 文件都是一个独立的 JS 文件，遵循统一模式：调用 `ProjectGuide.registerSection()`。

### 5.1 `js/sections/overview.js` — 项目概览

```javascript
/** Section: 项目概览 */
ProjectGuide.registerSection({
  id: 'overview',
  title: '项目概览',
  icon: '🏷️',
  priority: 10,
  render: function(container, data) {
    var p = data.project || {};
    var h = ProjectGuide.helpers;

    var html = '<div class="card">';
    html += '<div class="card-title">' + h.escapeHtml(p.name || '未命名') + '</div>';
    html += '<p><strong>描述：</strong>' + h.escapeHtml(p.description || '—') + '</p>';
    html += '<p><strong>目标用户：</strong>' + h.escapeHtml(p.targetUsers || '—') + '</p>';
    html += '<p><strong>成熟度：</strong>' + h.escapeHtml(p.maturity || '—') + '</p>';
    html += '</div>';

    // 技术栈标签云
    html += '<h3>🛠️ 技术栈</h3><div class="tag-cloud">';
    (data.techStack || []).forEach(function(t) {
      var tagClass = 'tag-blue';
      if (t.category === '框架') tagClass = 'tag-green';
      else if (t.category === '构建工具') tagClass = 'tag-yellow';
      else if (t.category === '包管理器') tagClass = 'tag-gray';
      html += '<span class="tag ' + tagClass + '" title="' + h.escapeHtml(t.evidence || '') + '">' +
              h.escapeHtml(t.tech) + '</span>';
    });
    html += '</div>';

    container.innerHTML = html;
  }
});
```

### 5.2 `js/sections/quickstart.js` — 快速开始

```javascript
/** Section: 快速开始 */
ProjectGuide.registerSection({
  id: 'quickstart',
  title: '快速开始',
  icon: '🚀',
  priority: 20,
  render: function(container, data) {
    var q = data.quickstart || {};
    var h = ProjectGuide.helpers;

    var html = '';

    // 命令项辅助函数
    function cmdBlock(label, command) {
      if (!command) return '';
      return '<div class="card" style="margin-bottom:8px">' +
        '<div style="font-weight:600;margin-bottom:6px">' + h.escapeHtml(label) + '</div>' +
        '<div style="position:relative;background:var(--code-bg);padding:10px 14px;border-radius:6px;font-family:monospace;font-size:13px">' +
        '<code>' + h.escapeHtml(command) + '</code>' +
        '<button class="copy-btn-inline" data-code="' + h.escapeHtml(command) + '" style="position:absolute;top:6px;right:8px;padding:2px 10px;font-size:12px;background:var(--bg-primary);border:1px solid var(--border);border-radius:4px;cursor:pointer;color:var(--text-secondary)" onclick="navigator.clipboard.writeText(this.getAttribute(\'data-code\'))">复制</button>' +
        '</div></div>';
    }

    if (q.requirements) {
      html += '<p><strong>环境要求：</strong>' + h.escapeHtml(q.requirements) + '</p>';
    }
    html += cmdBlock('安装依赖', q.install);
    html += cmdBlock('启动开发服务器', q.dev);
    html += cmdBlock('构建', q.build);
    html += cmdBlock('运行测试', q.test);

    if (q.urls && q.urls.length) {
      html += '<h4>访问地址</h4><ul>';
      q.urls.forEach(function(u) {
        html += '<li><strong>' + h.escapeHtml(u.label) + '：</strong>' +
                '<a href="' + h.escapeHtml(u.url) + '" target="_blank">' + h.escapeHtml(u.url) + '</a></li>';
      });
      html += '</ul>';
    }

    container.innerHTML = html;
  }
});
```

### 5.3 `js/sections/structure.js` — 项目结构树

```javascript
/** Section: 项目结构树 */
ProjectGuide.registerSection({
  id: 'structure',
  title: '项目结构树',
  icon: '📁',
  priority: 30,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var structure = data.structure || {};
    var tree = structure.tree || [];

    var html = '';

    // 全部展开/折叠按钮
    html += '<div class="tree-controls">' +
      '<button id="expandAll">📂 全部展开</button>' +
      '<button id="collapseAll">📁 全部折叠</button>' +
      '</div>';

    // 递归渲染目录树
    function renderTree(nodes, depth) {
      depth = depth || 0;
      var cls = depth === 0 ? 'tree-children' : 'tree-children';
      var style = depth <= 1 ? '' : 'style="display:none"';
      var out = '<div class="' + cls + '" ' + style + '>';
      nodes.forEach(function(node) {
        var hasChildren = node.children && node.children.length > 0;
        out += '<div class="tree-item">';
        if (hasChildren) {
          var open = depth <= 0;
          out += '<span class="tree-toggle">' + (open ? '▼' : '▶') + '</span>';
        } else {
          out += '<span class="tree-toggle" style="visibility:hidden">▶</span>';
        }
        out += '<span class="tree-label">';
        out += '<span class="tree-name">' + (node.icon || '📄') + ' ' + h.escapeHtml(node.name) + '</span>';
        if (node.priority) {
          var badgeClass = 'tree-badge ' + node.priority.toLowerCase();
          out += '<span class="' + badgeClass + '">' + h.escapeHtml(node.priority) + '</span>';
        }
        if (node.note) {
          out += '<span class="tree-note">' + h.escapeHtml(node.note) + '</span>';
        }
        out += '</span>';
        if (hasChildren) {
          out += renderTree(node.children, depth + 1);
        }
        out += '</div>';
      });
      out += '</div>';
      return out;
    }

    html += '<div class="tree-container">' + renderTree(tree) + '</div>';

    // 依赖方向
    if (structure.dependencyFlow) {
      html += '<h3>📊 依赖方向</h3>';
      html += '<div class="card"><code>' + h.escapeHtml(structure.dependencyFlow) + '</code></div>';
    }

    // 警告
    if (structure.warnings && structure.warnings.length) {
      html += '<h3>⚠ 结构警告</h3>';
      structure.warnings.forEach(function(w) {
        html += '<div class="card" style="border-left:3px solid var(--warning)">' +
                '<strong>' + h.escapeHtml(w.type) + '：</strong>' +
                h.escapeHtml(w.description) + '</div>';
      });
    }

    container.innerHTML = html;
  }
});
```

|

### 5.4 `js/sections/concept-map.js` — 概念地图 🧭

```javascript
/** Section: 概念地图 🧭 — 业务概念→代码位置的映射 */
ProjectGuide.registerSection({
  id: 'concept-map',
  title: '概念地图',
  icon: '🧭',
  priority: 35,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var cm = data.conceptMap || {};
    var concepts = cm.concepts || [];
    var html = '';

    // --- 说明 ---
    html += '<p style="margin-bottom:16px;color:var(--text-secondary)">' +
            '从业务概念出发理解项目。点击图中的概念节点或下方卡片，快速定位相关代码。' +
            '</p>';

    // --- Mermaid 概念关系图 ---
    if (cm.mermaidDef) {
      html += '<div class="mermaid-wrapper">';
      html += '<div class="mermaid">' + h.escapeHtml(cm.mermaidDef) + '</div>';
      html += '</div>';
    }

    // --- 概念详情卡片 ---
    if (concepts.length) {
      html += '<h3>📋 概念详情</h3>';
      html += '<div class="concept-grid">';

      concepts.forEach(function(c) {
        var badgeClass = (c.priority === 'P0') ? 'tag-red' :
                         (c.priority === 'P1') ? 'tag-yellow' : 'tag-gray';

        html += '<div class="card concept-card" id="concept-' + h.escapeHtml(c.id) + '">';

        // 名称 + 优先级
        html += '<div class="card-title">' + h.escapeHtml(c.name) +
                ' <span class="tag ' + badgeClass + '">' + h.escapeHtml(c.priority || '') + '</span>' +
                '</div>';

        // 描述
        if (c.description) {
          html += '<p style="margin-bottom:10px">' + h.escapeHtml(c.description) + '</p>';
        }

        // 关键文件列表
        if (c.files && c.files.length) {
          html += '<div style="font-size:13px"><strong>📂 关键文件：</strong></div>';
          html += '<ul style="margin:4px 0 10px;padding-left:18px;font-size:13px">';
          c.files.forEach(function(f) {
            html += '<li><a href="#section-structure" class="file-link" title="' + h.escapeHtml(f.desc || '') + '">' +
                    '<code>' + h.escapeHtml(f.path) + '</code>' +
                    (f.desc ? ' — ' + h.escapeHtml(f.desc) : '') +
                    '</a></li>';
          });
          html += '</ul>';
        }

        // 关联概念标签
        if (c.relatedConcepts && c.relatedConcepts.length) {
          html += '<div style="margin-top:8px">';
          html += '<span style="font-size:12px;color:var(--text-secondary)">🔗 关联：</span> ';
          c.relatedConcepts.forEach(function(rid) {
            var relatedConcept = concepts.find(function(x) { return x.id === rid; });
            var relatedName = relatedConcept ? relatedConcept.name : rid;
            html += '<a href="#concept-' + h.escapeHtml(rid) + '" class="related-tag">' +
                    h.escapeHtml(relatedName) + '</a> ';
          });
          html += '</div>';
        }

        html += '</div>'; // .concept-card
      });

      html += '</div>'; // .concept-grid
    } else {
      html += '<p style="color:var(--text-secondary)">（未识别业务概念）</p>';
    }

    container.innerHTML = html;

    // 初始化 Mermaid 图并添加点击事件
    if (cm.mermaidDef && typeof mermaid !== 'undefined') {
      try {
        mermaid.run({ nodes: [container.querySelector('.mermaid')] }).then(function() {
          var svg = container.querySelector('.mermaid svg');
          if (svg) {
            svg.style.cursor = 'default';
            concepts.forEach(function(c) {
              var textEls = svg.querySelectorAll('text');
              textEls.forEach(function(t) {
                if (t.textContent.indexOf(c.name) !== -1) {
                  var parentGroup = t.closest('g');
                  if (parentGroup) {
                    parentGroup.style.cursor = 'pointer';
                    parentGroup.addEventListener('click', function() {
                      var target = document.getElementById('concept-' + c.id);
                      if (target) {
                        target.scrollIntoView({ behavior: 'smooth', block: 'center' });
                        target.style.boxShadow = '0 0 0 3px var(--accent)';
                        setTimeout(function() { target.style.boxShadow = ''; }, 2000);
                      }
                    });
                  }
                }
              });
            });
          }
        }).catch(function() {});
      } catch(e) {}
    }
  }
});
```
### 5.5 `js/sections/architecture.js` — 架构概览

```javascript
/** Section: 架构概览 */
ProjectGuide.registerSection({
  id: 'architecture',
  title: '架构概览',
  icon: '🏗️',
  priority: 40,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var arch = data.architecture || {};
    var html = '';

    // Mermaid 架构图
    if (arch.mermaidDef) {
      html += '<div class="mermaid-wrapper">';
      html += '<div class="mermaid">' + h.escapeHtml(arch.mermaidDef) + '</div>';
      html += '</div>';
    }

    // 分层说明
    if (arch.layers && arch.layers.length) {
      html += '<h3>分层说明</h3>';
      arch.layers.forEach(function(layer) {
        html += '<div class="card">';
        html += '<div class="card-title">' + h.escapeHtml(layer.name) + '</div>';
        html += '<p>' + h.escapeHtml(layer.description || '') + '</p>';
        if (layer.files && layer.files.length) {
          html += '<p style="font-size:13px;color:var(--text-secondary)">' +
                  '关键文件：' + h.escapeHtml(layer.files.join(', ')) + '</p>';
        }
        html += '</div>';
      });
    }

    container.innerHTML = html;

    // 触发 Mermaid 渲染（如果有新图）
    if (arch.mermaidDef && typeof mermaid !== 'undefined') {
      try { mermaid.run({ nodes: [container.querySelector('.mermaid')] }); } catch(e) {}
    }
  }
});
```

### 5.6 `js/sections/code-analysis.js` — 代码展示与分析 ⭐（深层核心）

```javascript
/** Section: 代码展示与分析 ⭐ */
ProjectGuide.registerSection({
  id: 'code-analysis',
  title: '代码展示与分析',
  icon: '📝',
  priority: 50,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var snippets = data.codeSnippets || [];
    var html = '';

    if (!snippets.length) {
      html += '<p style="color:var(--text-secondary)">（未提取代码片段）</p>';
      container.innerHTML = html;
      return;
    }

    html += '<p style="margin-bottom:16px;color:var(--text-secondary)">' +
            '共提取 <strong>' + snippets.length + '</strong> 段关键业务代码，每段附带逐行分析。' +
            '</p>';

    snippets.forEach(function(snip, idx) {
      var fileLabel = snip.file;
      if (snip.lines) fileLabel += ':' + snip.lines;

      html += '<div class="code-panel" id="code-snippet-' + idx + '">';

      // 代码头部
      html += '<div class="code-header">';
      html += '<span class="file-icon">📄</span>';
      html += '<span class="file-path">' + h.escapeHtml(fileLabel) + '</span>';
      html += '<button class="copy-btn" data-code="' + h.escapeHtml(snip.code || '') + '" ' +
              'onclick="navigator.clipboard.writeText(this.getAttribute(\'data-code\'))">复制</button>';
      html += '</div>';

      // 代码内容
      html += '<pre class="code-content"><code>' + h.escapeHtml(snip.code || '') + '</code></pre>';

      // 分析注释
      var a = snip.analysis || {};
      html += '<div class="code-analysis">';
      html += '<div class="analysis-title">🔍 代码分析</div>';
      html += '<div class="analysis-body">';

      if (a.summary) {
        html += '<div class="analysis-field"><strong>功能：</strong>' + h.escapeHtml(a.summary) + '</div>';
      }

      if (a.lineNotes && a.lineNotes.length) {
        html += '<p style="margin-top:8px"><strong>逐行说明：</strong></p><ul>';
        a.lineNotes.forEach(function(n) {
          html += '<li><strong>' + h.escapeHtml(n.lines) + ':</strong> ' + h.escapeHtml(n.text) + '</li>';
        });
        html += '</ul>';
      }

      if (a.inputs) {
        html += '<div class="analysis-field"><strong>输入：</strong>' + h.escapeHtml(a.inputs) + '</div>';
      }
      if (a.outputs) {
        html += '<div class="analysis-field"><strong>输出：</strong>' + h.escapeHtml(a.outputs) + '</div>';
      }
      if (a.callers) {
        var callers = Array.isArray(a.callers) ? a.callers.join(', ') : a.callers;
        html += '<div class="analysis-field"><strong>调用方：</strong>' + h.escapeHtml(callers) + '</div>';
      }
      if (a.notes) {
        html += '<div class="note">注意：' + h.escapeHtml(a.notes) + '</div>';
      }

      html += '</div></div>'; // .analysis-body, .code-analysis
      html += '</div>'; // .code-panel
    });

    container.innerHTML = html;
  }
});
```

### 5.7 `js/sections/modules.js` — 核心模块详解

```javascript
/** Section: 核心模块详解 */
ProjectGuide.registerSection({
  id: 'modules',
  title: '核心模块详解',
  icon: '📦',
  priority: 60,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var modules = data.modules || [];
    var html = '';

    if (!modules.length) {
      html += '<p style="color:var(--text-secondary)">（未识别核心模块）</p>';
      container.innerHTML = html;
      return;
    }

    modules.forEach(function(mod) {
      var badgeClass = (mod.priority === 'P0') ? 'tag-red' :
                       (mod.priority === 'P1') ? 'tag-yellow' : 'tag-gray';
      html += '<details class="card" style="cursor:pointer">';
      html += '<summary style="font-weight:600;font-size:16px">' +
              h.escapeHtml(mod.name) +
              ' <span class="tag ' + badgeClass + '" style="margin-left:8px">' +
              h.escapeHtml(mod.priority || '') + '</span>' +
              '</summary>';
      html += '<div style="margin-top:12px">';
      html += '<p><strong>职责：</strong>' + h.escapeHtml(mod.responsibility || '—') + '</p>';

      if (mod.keyFiles && mod.keyFiles.length) {
        html += '<p><strong>关键文件：</strong>' +
                h.escapeHtml(mod.keyFiles.join(', ')) + '</p>';
      }
      if (mod.apis && mod.apis.length) {
        html += '<p><strong>对外 API：</strong>' +
                h.escapeHtml(mod.apis.join(', ')) + '</p>';
      }
      if (mod.dependencies && mod.dependencies.length) {
        html += '<p><strong>依赖：</strong>' +
                h.escapeHtml(mod.dependencies.join(', ')) + '</p>';
      }
      if (mod.risks && mod.risks.length) {
        html += '<p style="color:var(--danger)"><strong>⚠ 风险：</strong>' +
                h.escapeHtml(mod.risks.join('；')) + '</p>';
      }
      html += '</div></details>';
    });

    container.innerHTML = html;
  }
});
```

### 5.8 `js/sections/dataflow.js` — 数据流追踪

```javascript
/** Section: 数据流追踪 */
ProjectGuide.registerSection({
  id: 'dataflow',
  title: '数据流追踪',
  icon: '🔄',
  priority: 70,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var df = data.dataflow || {};
    var html = '';

    // Mermaid 时序图
    if (df.mermaidDef) {
      html += '<div class="mermaid-wrapper">';
      html += '<div class="mermaid">' + h.escapeHtml(df.mermaidDef) + '</div>';
      html += '</div>';
    }

    // 逐步说明
    if (df.steps && df.steps.length) {
      html += '<h3>关键步骤</h3>';
      html += '<table><thead><tr><th>#</th><th>参与者</th><th>动作</th><th>位置</th></tr></thead><tbody>';
      df.steps.forEach(function(s) {
        html += '<tr>' +
                '<td>' + s.order + '</td>' +
                '<td>' + h.escapeHtml(s.actor || '') + '</td>' +
                '<td>' + h.escapeHtml(s.action || '') + '</td>' +
                '<td><code>' + h.escapeHtml(s.location || '') + '</code></td>' +
                '</tr>';
      });
      html += '</tbody></table>';
    }

    container.innerHTML = html;

    // 渲染 Mermaid 图
    if (df.mermaidDef && typeof mermaid !== 'undefined') {
      try { mermaid.run({ nodes: [container.querySelector('.mermaid')] }); } catch(e) {}
    }
  }
});
```

### 5.9 `js/sections/tracking.js` — 埋点事件清单

```javascript
/** Section: 埋点事件清单 */
ProjectGuide.registerSection({
  id: 'tracking',
  title: '埋点事件清单',
  icon: '📊',
  priority: 80,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var t = data.tracking || {};
    var events = t.events || [];
    var html = '';

    // SDK 信息
    if (t.sdks && t.sdks.length) {
      html += '<p><strong>识别到的埋点 SDK：</strong>';
      html += t.sdks.map(function(s) { return '<span class="tag tag-blue">' + h.escapeHtml(s) + '</span>'; }).join(' ');
      html += '</p>';
    }

    // 搜索 + 表格
    if (events.length) {
      html += '<div class="table-search">' +
              '<input type="text" class="tracking-search" placeholder="搜索埋点事件..." ' +
              'oninput="var q=this.value.toLowerCase();' +
              'this.parentElement.nextElementSibling.querySelectorAll(\'tbody tr\').forEach(function(r){' +
              'r.style.display=r.textContent.toLowerCase().indexOf(q)===-1?\'none\':\'\'})">' +
              '</div>';

      html += '<table><thead><tr>' +
              '<th>事件名</th><th>类别</th><th>触发时机</th><th>位置</th><th>参数</th><th>状态</th>' +
              '</tr></thead><tbody>';

      events.forEach(function(e) {
        var statusTag = (e.status === 'confirmed') ? '✅' :
                        (e.status === 'inferred') ? '🔍' : '❓';
        html += '<tr>' +
                '<td><code>' + h.escapeHtml(e.eventName) + '</code></td>' +
                '<td>' + h.escapeHtml(e.category || '') + '</td>' +
                '<td>' + h.escapeHtml(e.trigger || '') + '</td>' +
                '<td><code>' + h.escapeHtml(e.location || '') + '</code></td>' +
                '<td style="font-size:13px">' + h.escapeHtml(e.params || '') + '</td>' +
                '<td>' + statusTag + '</td>' +
                '</tr>';
      });

      html += '</tbody></table>';
    } else {
      html += '<p style="color:var(--text-secondary)">（未识别到埋点事件）</p>';
    }

    // 覆盖缺口
    if (t.gaps && t.gaps.length) {
      html += '<h3>⚠ 覆盖缺口</h3><table><thead><tr><th>流程</th><th>应有埋点</th><th>现状</th><th>建议</th></tr></thead><tbody>';
      t.gaps.forEach(function(g) {
        html += '<tr>' +
                '<td>' + h.escapeHtml(g.flow) + '</td>' +
                '<td>' + h.escapeHtml(g.expected || '') + '</td>' +
                '<td>' + h.escapeHtml(g.actual || '') + '</td>' +
                '<td>' + h.escapeHtml(g.suggestion || '') + '</td>' +
                '</tr>';
      });
      html += '</tbody></table>';
    }

    container.innerHTML = html;
  }
});
```

### 5.10 `js/sections/workflow.js` — 开发工作流

```javascript
/** Section: 开发工作流 */
ProjectGuide.registerSection({
  id: 'workflow',
  title: '开发工作流与约定',
  icon: '🔧',
  priority: 90,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var w = data.workflow || {};
    var html = '';

    var items = [
      { label: '分支策略', value: w.branching },
      { label: '命名规范', value: w.conventions && w.conventions.naming },
      { label: '提交规范', value: w.conventions && w.conventions.commit },
      { label: '代码风格', value: w.conventions && w.conventions.codeStyle },
      { label: 'Code Review', value: w.review },
      { label: 'CI/CD', value: w.ci }
    ];

    html += '<table><thead><tr><th>项目</th><th>说明</th></tr></thead><tbody>';
    items.forEach(function(item) {
      if (item.value) {
        html += '<tr><td><strong>' + item.label + '</strong></td><td>' + h.escapeHtml(item.value) + '</td></tr>';
      }
    });
    html += '</tbody></table>';

    container.innerHTML = html;
  }
});
```

### 5.11 `js/sections/roadmap.js` — 新人阅读路线

```javascript
/** Section: 新人阅读路线 */
ProjectGuide.registerSection({
  id: 'roadmap',
  title: '新人阅读路线',
  icon: '🗺️',
  priority: 100,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var rm = data.roadmap || {};
    var html = '';

    function renderRole(title, steps) {
      if (!steps || !steps.length) return '';
      var out = '<h3>' + h.escapeHtml(title) + '</h3><ol>';
      steps.forEach(function(s) {
        out += '<li><strong>' + h.escapeHtml(s.action) + '</strong>' +
               (s.goal ? ' — ' + h.escapeHtml(s.goal) : '') + '</li>';
      });
      out += '</ol>';
      return out;
    }

    html += '<p style="color:var(--text-secondary);margin-bottom:16px">按角色推荐阅读顺序，从浅入深逐步理解项目。</p>';
    html += renderRole('🎨 前端开发者', rm.frontend);
    html += renderRole('⚙️ 后端开发者', rm.backend);
    html += renderRole('🚀 全栈开发者', rm.fullstack);

    container.innerHTML = html;
  }
});
```

### 5.12 `js/sections/risks.js` — 风险和注意事项

```javascript
/** Section: 风险和注意事项 */
ProjectGuide.registerSection({
  id: 'risks',
  title: '风险和注意事项',
  icon: '⚠️',
  priority: 110,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var risks = data.risks || [];
    var html = '';

    if (!risks.length) {
      html += '<p style="color:var(--text-secondary)">（未识别显著风险）</p>';
      container.innerHTML = html;
      return;
    }

    html += '<table><thead><tr><th>严重度</th><th>描述</th><th>证据</th><th>影响</th><th>建议</th></tr></thead><tbody>';
    risks.forEach(function(r) {
      var sevBadge = '';
      if (r.severity === 'high') sevBadge = '<span class="tag tag-red">高</span>';
      else if (r.severity === 'medium') sevBadge = '<span class="tag tag-yellow">中</span>';
      else sevBadge = '<span class="tag tag-gray">低</span>';

      html += '<tr>' +
              '<td>' + sevBadge + '</td>' +
              '<td>' + h.escapeHtml(r.description || '') + '</td>' +
              '<td><code>' + h.escapeHtml(r.evidence || '') + '</code></td>' +
              '<td>' + h.escapeHtml(r.impact || '') + '</td>' +
              '<td>' + h.escapeHtml(r.suggestion || '') + '</td>' +
              '</tr>';
    });
    html += '</tbody></table>';

    container.innerHTML = html;
  }
});
```

### 5.13 `js/sections/appendix.js` — 附录

```javascript
/** Section: 附录 */
ProjectGuide.registerSection({
  id: 'appendix',
  title: '附录',
  icon: '📋',
  priority: 120,
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var ap = data.appendix || {};
    var html = '';

    // 常用命令
    if (ap.commands && ap.commands.length) {
      html += '<h3>常用命令速查</h3><table><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody>';
      ap.commands.forEach(function(c) {
        html += '<tr><td><code>' + h.escapeHtml(c.name) + '</code></td><td>' + h.escapeHtml(c.description || '') + '</td></tr>';
      });
      html += '</tbody></table>';
    }

    // 环境变量
    if (ap.envVars && ap.envVars.length) {
      html += '<h3>环境变量</h3><table><thead><tr><th>变量名</th><th>说明</th><th>默认值</th></tr></thead><tbody>';
      ap.envVars.forEach(function(v) {
        html += '<tr><td><code>' + h.escapeHtml(v.name) + '</code></td>' +
                '<td>' + h.escapeHtml(v.description || '') + '</td>' +
                '<td><code>' + h.escapeHtml(v.defaultValue || '—') + '</code></td></tr>';
      });
      html += '</tbody></table>';
    }

    // 名词解释
    if (ap.glossary && ap.glossary.length) {
      html += '<h3>名词解释</h3><table><thead><tr><th>术语</th><th>说明</th></tr></thead><tbody>';
      ap.glossary.forEach(function(g) {
        html += '<tr><td><strong>' + h.escapeHtml(g.term) + '</strong></td><td>' + h.escapeHtml(g.definition || '') + '</td></tr>';
      });
      html += '</tbody></table>';
    }

    container.innerHTML = html;
  }
});
```

---

## 6. 扩展开发指南：`extensions/README.md`

```markdown
# 扩展 Project Guide

本目录是 project-guide 的扩展点。你可以在此添加新的分析维度，无需修改核心框架。

## 三步添加新 Section

### 步骤 1：添加数据

在 `js/guide-data.js` 的 `window.__PROJECT_DATA__` 对象中添加新字段：

\`\`\`js
window.__PROJECT_DATA__ = {
  // ... 现有字段 ...
  
  // 新增：API 文档分析数据
  apiDocs: [
    { endpoint: 'GET /api/users', description: '获取用户列表', params: '...', response: '...' }
  ]
};
\`\`\`

### 步骤 2：创建 Section 渲染器

在 `js/sections/` 下新建文件（如 `api-docs.js`）：

\`\`\`js
/** Section: API 文档 */
ProjectGuide.registerSection({
  id: 'api-docs',
  title: 'API 文档',
  icon: '🔌',
  priority: 55,  // 控制排序位置
  render: function(container, data) {
    var h = ProjectGuide.helpers;
    var apis = data.apiDocs || [];
    
    var html = '<table>' +
      '<thead><tr><th>端点</th><th>说明</th><th>参数</th><th>响应</th></tr></thead><tbody>';
    
    apis.forEach(function(api) {
      html += '<tr>' +
        '<td><code>' + h.escapeHtml(api.endpoint) + '</code></td>' +
        '<td>' + h.escapeHtml(api.description) + '</td>' +
        '<td>' + h.escapeHtml(api.params || '—') + '</td>' +
        '<td>' + h.escapeHtml(api.response || '—') + '</td>' +
        '</tr>';
    });
    
    html += '</tbody></table>';
    container.innerHTML = html;
  }
});
\`\`\`

### 步骤 3：注册到入口

在 `index.html` 中添加一行：

\`\`\`html
<script src="js/sections/api-docs.js"></script>
\`\`\`

**完成！** 无需改动 `guide-core.js` 或任何现有 section。

## Section 配置项说明

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | ✅ | 唯一标识，用作 DOM id（`section-{id}`）和导航锚点 |
| `title` | string | ✅ | 侧边栏导航中显示的名称 |
| `icon` | string | 推荐 | 导航中的 emoji 图标 |
| `priority` | number | 推荐 | 排序权重，越小越靠前。参考值：10=概览, 50=代码分析, 120=附录 |
| `render` | function | ✅ | `function(container, data)` — container 是 DOM 元素，data 是 `__PROJECT_DATA__` |
| `onNavClick` | function | 可选 | 点击导航时的自定义行为，默认滚动到 section 位置 |

## 扩展灵感

- 🧭 **概念地图**：业务概念 → 代码位置的可视化映射（Mermaid 关系图 + 详情卡片）
- 🔌 **API 文档**：Swagger/OpenAPI 规格展示
- 🗄️ **数据库 Schema**：表结构 ER 图（Mermaid）
- ✅ **测试覆盖率**：测试统计和覆盖率可视化
- 📦 **依赖分析**：npm/maven/pip 依赖树与版本信息
- 🚢 **部署拓扑**：服务器/容器/服务关系图
- 🔐 **权限矩阵**：角色-权限对照表
- 📈 **性能基准**：关键指标和基准数据
- 🧩 **组件 Storybook**：组件变体展示（前端项目）
```

---

## 7. 生成流程（AI Agent 指引）

Agent 在执行深层分析时按以下步骤生成文件：

1. **创建目录结构**
   ```bash
   mkdir -p project-guide/css
   mkdir -p project-guide/js/sections
   mkdir -p project-guide/extensions
   ```

2. **按顺序写入文件**
   | 顺序 | 文件 | 说明 |
   | --- | --- | --- |
   | 1 | `css/guide.css` | 复制上述 CSS 模板，无需修改 |
   | 2 | `js/guide-core.js` | 复制上述核心框架模板，无需修改 |
   | 3 | `js/guide-data.js` | **根据分析结果填充数据** |
   | 4-16 | `js/sections/*.js` | 每个 section 文件使用上述模板，数据字段即为 `data.xxx` |
   | 17 | `index.html` | 复制上述模板，替换 `{项目名称}` 和 `{生成日期}` |
   | 18 | `extensions/README.md` | 复制上述扩展指南 |

3. **告知用户**
   > 已生成 `project-guide/` 目录，包含交互式项目指南。
   > 用浏览器打开 `project-guide/index.html` 即可查看。

---

## 8. 与聚焦分析 HTML 的关系

聚焦分析（选项 3）选择 html 格式输出时，有两种策略：

**策略 A（推荐）：复用模块化架构**

生成 `project-guide/{模块名}-focus/` 目录，结构与深层分析相同但只有模块相关的 section（精简为 5-6 个 section）。复用相同的 `guide-core.js` 和 `guide.css`。

**策略 B：单文件 HTML（简单场景）**

如果模块较小、分析内容较少，可以回退到单文件 `{模块名}-focus.html`，使用聚焦分析独有模板（代码分析更细致，逐文件完整分析）。

策略选择由 Agent 根据模块规模判断：文件数 > 10 或代码量 > 2000 行时建议策略 A。

---

## 注意事项

- **文件总数**：深层分析生成约 18 个文件，总体积通常 < 300KB（不含 Mermaid CDN）
- **Mermaid CDN**：作为可选依赖，无网络时图表区降级为 `<pre>` 静态文本
- **数据安全**：guide-data.js 中不得包含密钥、Token、连接串等敏感信息
- **路径兼容**：所有引用使用相对路径（`js/...`、`css/...`），兼容 `file://` 协议
- **Section 隔离**：一个 section 渲染失败不影响其他 section（内部 try-catch）
- **删除 Section**：若要移除某个章节，只需删除对应的 `<script>` 标签和 section 文件
