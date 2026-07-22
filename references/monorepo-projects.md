# Monorepo 项目分析

适用于使用 monorepo 架构的项目（多个子项目/包在同一仓库中）。

## 关键问题

- 使用什么 monorepo 工具？（Turborepo / Nx / Lerna / Rush / pnpm workspaces / Yarn workspaces / Bazel）
- 有哪些子项目/包？各自职责是什么？
- 子项目之间的依赖关系是怎样的？
- 共享代码如何组织？
- 构建和测试如何编排？
- 版本管理策略是什么？
- CI/CD 如何处理增量构建？

## 盘点步骤

### 1. 识别 Monorepo 工具

```bash
# 检查 monorepo 工具特征
rg -l "workspaces" package.json pnpm-workspace.yaml lerna.json nx.json turbo.json rush.json 2>/dev/null
```

| 特征文件 | 工具 |
| --- | --- |
| `pnpm-workspace.yaml` | pnpm workspaces |
| `lerna.json` | Lerna |
| `nx.json` | Nx |
| `turbo.json` | Turborepo |
| `rush.json` | Rush |
| `WORKSPACE` (Bazel) | Bazel |
| `package.json` 中含 `"workspaces"` | Yarn/npm workspaces |

### 2. 子项目清单

对每个子项目生成登记表：

| 子项目 | 路径 | 类型 | 职责 | 技术栈 | 依赖兄弟包 |
| --- | --- | --- | --- | --- | --- |
| `@org/web` | `apps/web/` | 前端应用 | 用户端 | Next.js, React | `@org/ui`, `@org/utils` |
| `@org/api` | `apps/api/` | 后端服务 | API 服务 | Express, TypeScript | `@org/utils` |
| `@org/ui` | `packages/ui/` | 共享组件库 | UI 组件 | React, Storybook | `@org/types` |
| `@org/utils` | `packages/utils/` | 工具库 | 通用函数 | TypeScript | — |
| `@org/types` | `packages/types/` | 类型定义 | 共享类型 | TypeScript | — |

### 3. 依赖关系图

```text
apps/web ──────► packages/ui ──────► packages/types
     │
     └──────────► packages/utils ──► packages/types

apps/api ───────► packages/utils ──► packages/types
```

### 4. 共享代码分析

- 共享包是否被合理使用（不包含业务逻辑）？
- 共享包是否有独立的测试和文档？
- 是否存在"万能 utils"包（所有东西都往里放）？
- 共享包版本是否锁定？

## 子项目分析策略

对每个子项目独立分析，然后合并结果：

1. **逐个分析**：将每个子项目视为独立项目，执行完整的分析流程
2. **共享部分**：共享包单独分析，关注其 API 和版本
3. **交叉引用**：
   - 谁依赖了这个共享包？依赖了哪些导出？
   - 这个子项目的 API 被哪个子项目调用？
4. **合并报告**：生成统一的 monorepo 总览 + 各子项目详情

## 构建和任务编排

### Turborepo 项目
- `turbo.json` 中的 pipeline 定义
- 任务依赖关系（dependsOn）
- 缓存策略（inputs/outputs）

### Nx 项目
- `nx.json` 中的任务编排
- 项目依赖图（`nx graph`）
- 生成器（generators）和执行器（executors）

### pnpm workspaces 项目
- `pnpm-workspace.yaml` 中的工作空间定义
- 根 `package.json` 的脚本如何编排子项目
- 过滤执行：`pnpm --filter @org/web build`

## 版本管理

- 使用固定版本（所有包同一版本）还是独立版本？
- 是否使用 Changesets / Auto / semantic-release？
- 发布流程：CI 自动发布 vs 手动发布

## CI/CD 注意事项

- 是否有增量构建（只构建变更的子项目）？
- 缓存如何跨 CI 运行共享？
- 测试是否支持按子项目过滤？
- 部署是否按子项目独立部署？
- 环境变量如何按子项目隔离？

## 常见失败模式

- 共享包修改后忘记重新构建，依赖方使用旧版本
- 循环依赖导致构建失败
- 子项目间的 API 契约不明确
- 版本升级时破坏性变更未及时沟通
- 根目录和子项目的依赖版本冲突
- CI 全量构建耗时过长
- 共享包中混入业务逻辑
- `.gitignore` 配置不当导致构建产物互相影响
- 环境变量在各子项目间泄漏或混淆

## 推荐验证

- 确认根目录的 dev/build/test 命令能正常运行
- 检查每个子项目的独立构建是否成功
- 验证子项目间依赖是否一致（同一包的版本是否统一）
- 查看 CI 配置确认增量构建逻辑
- 检查是否有子项目未在 CI 中覆盖

## Monorepo HTML 指南特殊处理

生成深层 HTML 指南时：

- 结构树统一展示，子项目作为一级节点
- 每个子项目有独立的章节（概览、结构、模块、埋点）
- 共享包在导航中单独分组
- 依赖关系图使用 Mermaid 渲染
- 每个子项目的代码展示独立成节
