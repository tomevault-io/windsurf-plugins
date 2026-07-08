---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库当前状态(2026-07-02)

§5.1–§5.7 全章节 + §11 Q6(BullMQ + Redis + Bull-Board)+ §11 Q7(多 Skill Bundle 并存 + replay-with-latest)+ §11 Q13(代理 / git 凭证)+ §5.4 多 ScanRun 对比 + §5.3 API 覆盖端到端 + §6.2 首次登录改密码 + §6.2 真 JWT 解码 + AdminGuard + §5.4 报告 Markdown 渲染 + §5.7 真接 git clone + §5.7 真接 from-github(GitHub REST tarball)+ §11 Q11 Phase 4 Docker 化部署(3 服务 multi-stage build)+ CI/CD pipeline + Vitest coverage v8 provider + thresholds 强制门禁(api 覆盖率 80.18%,已启用且通过)全部落地。**本会话 38 commit(`952452d` → `5da5849`),454 测试通过(shared 6 / api 405 / web 43),0 错 0 警,typecheck 全绿**。本会话第三轮 commit(`3b033b2`→`5da5849`):VulnLibrary tab 真 tab + API 覆盖统计(COMPLETE)+ 核实 ScanDiff 已实现 + MetricsService DI 补缺 + 迁移补偿 + start.bat 自动关窗 + proxy 不保存即测 + AI Key Create 按钮可用 + 413 body-parser + Upload JSON 容错 + AI Key 扫描选择 + Proxy 自定义测试 + bat 脚本 CRLF + 上传 ValidationPipe root cause + ConfigPage proxy 按钮去重 + Skill Bundle 下拉 + Overview 版本列表。端到端可跑通:

- 上传 zip → 触发 BullMQ + Redis 队列 → 115 秒 → 漏洞入库 + 自动聚合到 VulnLibraryEntry
- 多 ScanRun 对比(端到端实测,DB 真写 + redis 真跑 + diff 端点返回完整 ScanDiff)
- API 覆盖统计(recompute-coverage 端点 + DB 真写 PARTIAL/COMPLETE)
- Members grant/revoke/role change(只 owner / lead 能改)
- git 凭证 + 代理 + 真接 git clone(`POST /api/code-versions/from-git` + 错误分类:NO_CREDENTIAL / AUTH_FAILED / AUTH_FORBIDDEN / NETWORK_UNREACHABLE / TIMEOUT / DISK_FULL / GIT_NOT_FOUND)
- **真接 GitHub REST tarball API**(`POST /api/code-versions/from-github` + `GitHubService.downloadTarball` + 自写 minimal tar 解压;凭证优先级 env GITHUB_TOKEN > git_credentials;错误分类:AUTH_FAILED / AUTH_FORBIDDEN / NOT_FOUND / RATE_LIMITED / SERVER_ERROR / NETWORK_UNREACHABLE / TIMEOUT)
- 改密码(§6.2 落地)
- 真 JWT 解码 + AdminGuard(替换 x-user-id mock,`JwtStrategy` + `JwtAuthGuard` + `RolesGuard` + `@Roles('admin')` 拦截 AI Key / Git Credentials / Proxy / Users 写端点)
- Bull-Board 队列可视化(`/admin/queue`,JWT admin OR Basic admin/admin 双通道鉴权,挂 express middleware 不走 NestJS 路由)
- 多 Skill Bundle 并存 + Replay (Latest Skill) 按钮(`POST /api/scan-runs/:id/replay-with-latest` 拿 `getDefault()` bundle 重跑)
- CI pipeline(`.github/workflows/ci.yml`,8 step,PR 自动跑 typecheck/test/lint + coverage v8 artifact)
- 报告 Markdown 渲染(react-markdown + remark-gfm + rehype-highlight + 章节导航 IntersectionObserver)

**2026-07-02 前端 UI 重设计**(3 commit:`d686123` → `c704126`):靛蓝主题(243 75% 59%)+ Light/Dark 双套 + 全域毛玻璃(glass-surface/glass-card/glass-popover)+ 三区布局(TopBar+Sidebar+Content 可折叠)+ Inter+JetBrains Mono 字体 + 14 个 shadcn/ui 组件 + 9 个业务组件(SeverityBadge/StatusBadge/StatCard/EmptyState/PageHeader/ThemeProvider/ThemeToggle/TopBar/Sidebar)+ sonner Toast + 13 页面全部组件化重构。42 测试全过,typecheck 全绿,lint 0 错误。

根目录含:

- `./需求文档.md` —— 1,418 行的产品/技术规格,锁定了 Q1–Q17 共 17 项决策
- `./dotnet-security-audit-skill/` —— **独立 git 仓库**(独立 .git/、独立 main 分支),内含 38 个 .NET 审计 skill + 主 agent.md + 9 份 shared 规范;平台不修改它
- `./apps/api/` —— **NestJS 后端**(modules: admin/queue-board / agents / auth / code-versions / db / git-clone / health / projects / realtime / report / scan / settings / skill-bundles / storage / users / vulns,共 16 个 + db 1)
- `./apps/web/` —— **React + Vite + shadcn/ui 前端**(路由: /login / /projects / /projects/:id / /projects/:id/scans/:runId / /projects/:id/scans/:runId/report / /projects/:id/vuln-library / /projects/:id/vuln-library/:libId / /admin/users / /admin/config)
- `./packages/shared/` —— 跨 api/web 共享的枚举与类型(严格对应 §4.2 / §11)
- `./pnpm-workspace.yaml` + `./package.json` + `./tsconfig.base.json` —— pnpm workspace + TS / ESLint / Prettier / Vitest 全栈配置
- `./eslint.config.js` + `./.prettierrc.json` + `./vitest.config.ts` —— 跨包统一代码风格
- `./start.bat` + `./stop.bat` + `./status.bat` —— Windows 快捷脚本(双击执行)

## 核心技术栈(锁定)

| 角色 | 选型 |
|------|------|
| AI 编排 | `@openai/agents`(OpenAI Agents SDK, TS/JS)+ `openai` SDK(实际跑通) |
| 后端 | NestJS 10 + TypeScript 5.7 |
| 前端 | React 18 + Vite 5.4 + shadcn/ui(15 组件)+ Tailwind CSS 3 · 靛蓝主题(243 75% 59%)+ Light/Dark + 毛玻璃 + Inter/JetBrains Mono |
| 数据库 | SQLite 3.x + Drizzle ORM(MVP) |
| 包管理 | pnpm 10(workspace) |
| 测试 | Vitest 2(shared/api/web 三个 project) |
| Lint | ESLint 9(flat config)+ Prettier 3 |
| 鉴权 | `@nestjs/jwt` + `@nestjs/passport` + `passport-jwt`(`JwtAuthGuard` + `RolesGuard` + `@Roles(...)`)|
| 运行时 | Node.js ≥ 20 LTS(本机 24.14.1) |

## Monorepo 布局

```
.
├── apps/
│   ├── api/        # @platform/api  —— NestJS 14 modules
│   │   └── src/
│   │       ├── agents/        # @openai/agents loader + PoC
│   │       ├── auth/          # JWT + argon2id 登录
│   │       ├── code-versions/ # §5.2 zip 上传 + SHA-256 + LOC + §5.7 from-git
│   │       ├── scan/          # §5.3 ScanModule + Runner + tools + processor
│   │       ├── report/        # §5.4 Markdown/JSON/zip
│   │       ├── vulns/         # §5.5 VulnLibraryService + VulnService
│   │       ├── projects/      # §5.1 CRUD
│   │       ├── users/         # 用户管理
│   │       ├── settings/      # AI Key(AES-256-GCM 加密)+ Git Credentials + Proxy
│   │       ├── skill-bundles/ # SkillBundleVersion 只读 + setDefault/publish
│   │       ├── storage/       # 路径工具
│   │       ├── realtime/      # WebSocket Gateway
│   │       ├── health/        # /api/health

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZMR0zhangmouren/DOT.NET-Code-Security-Audit-Platform](https://github.com/ZMR0zhangmouren/DOT.NET-Code-Security-Audit-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
