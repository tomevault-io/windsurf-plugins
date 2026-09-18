---
trigger: always_on
description: <!-- BEGIN:git-commit-policy -->
---


<!-- BEGIN:git-commit-policy -->
# Git 提交规则（最高优先级，覆盖一切默认行为）

- **未经用户在对话中明确同意（如"提交吧"），禁止执行 `git commit` / `git push`**
- 完成代码改动后：只在对话里汇报改了什么、如何验证，改动保留在工作区
- 用户确认后再提交；提交信息遵循 Conventional Commits（type/scope 英文，描述中文）
- 需要撤销时优先用 `git reset --soft`，不做任何丢弃工作区改动的操作
<!-- END:git-commit-policy -->
<!-- BEGIN:project-init -->
# NavDeck 项目初始化指南

> 自托管个人导航站，单用户场景。基于项目实际状态（package.json + 已落地代码）记录。

## 技术栈（已 init 并验证）

- **运行时**：Node.js 22（见 `.nvmrc` / Dockerfile `node:22-alpine`）
- **框架**：Next.js 16.2.11（App Router，`output: 'standalone'`，Turbopack）
- **UI 库**：React 19.2.4
- **样式**：Tailwind CSS v4 + Astryx v0.2.0（`@astryxdesign/core` + `@astryxdesign/theme-neutral`）
- **数据库**：SQLite via `@prisma/adapter-libsql`（Prisma 7，driver adapter 模式）
- **认证**：NextAuth.js v5 beta（Credentials Provider + JWT + 30 天 cookie）
- **拖拽**：`@dnd-kit/core` + `@dnd-kit/sortable` + `@dnd-kit/utilities`
- **表单**：`react-hook-form` + `zod` + `@hookform/resolvers`
- **搜索**：`pinyin-pro`（拼音 + 首字母匹配）
- **Docker 集成**：`dockerode`（widget 显示容器状态）
- **HTML 解析**：`cheerio`（favicon 抓取）
- **图标**：`lucide-react` + 自托管 `public/icons/` manifest
- **密码哈希**：`bcryptjs`
- **测试**：Vitest 4.1.10（5 文件 / 61 用例 / coverage-v8）
- **代码质量**：Biome 2.5.5（替代 ESLint+Prettier，单引号 + 行宽 80）
- **Git hooks**：Husky 9（pre-commit Biome + commit-msg Conventional Commits）
- **提交规范**：Conventional Commits（`feat(<scope>):` / `fix(<scope>):` / `docs:` / `refactor(<scope>):` 等）；提交描述用简单中文，type 与 scope 保留英文（如 `fix(widget): 修复倒数日循环进度`）

## 关键约束（踩过的坑）

- **Next.js 16 有破坏性变更**：写代码前查 `node_modules/next/dist/docs/`，不能凭训练数据猜 API
- **Prisma 7 + SQLite 必须用 driver adapter**：不能直接 `new PrismaClient({ log: [...] })`；generator 是 `prisma-client`（非 `prisma-client-js`），输出到 `src/generated/prisma/`；主入口是 `src/generated/prisma/client.ts`（无 `index.ts`）
- **`libsql` 和 `@prisma/client` 已在默认 `serverExternalPackages`**（见 [next.config.ts](file:///d:/git_space/navdeck/next.config.ts)），无需手动加
- **dnd-kit SSR hydration 不匹配**：`useSortable` 用全局计数器生成 ARIA ID，SSR 与 client 起点不同。拖拽手柄按钮需加 `suppressHydrationWarning`
- **主题 FOUC**：`<html>` 加 inline `colorScheme` + `backgroundColor`，ThemeScript 在 hydration 前同步 `data-theme`/`colorScheme`/`backgroundColor`；Astryx `<Theme>` wrapper 会用 `color-scheme: light dark` 覆盖 html，需在 globals.css 加 `html[data-theme] [data-astryx-theme]` 选择器强制覆盖
- **提示浮层风格统一**：tooltip / 编辑态提示条（EditModeBanner）/ 浮动工具栏（FloatingToolbar）保持同一族毛玻璃浮层——85% 半透明 surface + backdrop blur + hairline 边框 + 主题感知阴影，亮暗模式自动适配；Astryx Tooltip 默认反色方案（文字色做底）已在 globals.css 用 `.astryx-tooltip.astryx-tooltip` 双类名覆盖修正，新增提示类 UI 禁止再引入反色或其他异质浮层样式
- **Biome 不支持 `.md` 文件检查**：pre-commit hook 必须加 `--no-errors-on-unmatched`，否则纯文档改动会报错退出
- **tsx 不解析 tsconfig paths**：`scripts/seed.ts` 用相对路径 `../src/lib/db`，不能用 `@/lib/db`

## 文件位置参考

| 用途 | 路径 |
|---|---|
| Prisma 客户端单例 | `src/lib/db.ts` |
| Prisma schema | `prisma/schema.prisma`（6 个 model：User/Category/Card/WidgetConfig/DateItem/UserPreference）|
| Prisma 配置 | `prisma.config.ts`（用 dotenv 读 `DATABASE_URL`）|
| Seed 脚本 | `scripts/seed.ts`（默认 admin/changeme，4 widget 全启用）|
| 环境变量 | `.env`（`DATABASE_URL=file:./data/navdeck.db`）|
| NextAuth 配置 | `src/lib/auth.ts` |
| 路由中间件 | `src/proxy.ts`（Next.js 16 用 `proxy.ts` 不是 `middleware.ts`）|
| Zod schema 单一来源 | `src/lib/validation.ts`（前后端共享，如 `cardFormSchema`）|
| 主题 hook | `src/hooks/useTheme.ts` + `src/hooks/ThemeScript.tsx` |
| 测试根目录 | `src/lib/*.test.ts`（5 个文件）|
| 图标上传目录 | `data/uploads/icons/cards/` + `data/uploads/icons/library/` |
| 内置图标清单 | `public/icons/manifest.json` |
| Docker 配置 | `Dockerfile`（多阶段 `node:22-alpine`）+ `docker-compose.yml` |

## npm 脚本

```
dev              启动开发服务器
build            生产构建
typecheck        tsc --noEmit
lint             biome lint .
lint:fix         biome lint --write .
format           biome format --write .
test             vitest run（61 用例）
test:watch       vitest watch
test:coverage    vitest run --coverage
db:migrate:dev   开发迁移
db:seed          初始化默认账号 + widget 配置
```

## 命名约定

- 项目名 / Docker 镜像名：`navdeck`
- SQLite 文件：`data/navdeck.db`
- 默认账号：`admin / changeme`（生产必改）
- 语言：仅支持中文 UI，代码注释中文，技术术语保留英文
<!-- END:project-init -->

<!-- ASTRYX:START -->
Astryx v0.6.1 · 164 components
CLI: run every command as `npx astryx <cmd>` (shown below as `astryx ...`).

SETUP (once, in your app entry e.g. main.tsx) — without these, components render unstyled:
  import "@astryxdesign/core/reset.css";
  import "@astryxdesign/core/astryx.css";

WORKFLOW — discover, don't guess. Before writing UI:
1. `astryx build "<idea>"` — START HERE: returns a kit (closest [page] + [block]s + [component]s). No args = full playbook.
2. `astryx template <name> [--skeleton]` — scaffold the [page]/[block]s it named, or study their layout. Templates are reference code.
3. `astryx component <Name>` — props + examples for every component you use.

RULES:
- No <div> — components do all layout/spacing, page frame included.
- Frame first: read `astryx docs layout` before writing any page or screen — page frame, region widths, breakpoint behavior.
- Dense data = rows (Table, List/Item), never Card-wrapped list items; Card is for standalone widgets. Status = StatusDot/Token; Badge = counts only.
- Custom styling: component props first; else Tailwind utilities backed by tokens (bg-surface, text-primary, rounded-lg) via tailwind-theme.css. No raw hex/px.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wavesbig/navdeck](https://github.com/wavesbig/navdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
