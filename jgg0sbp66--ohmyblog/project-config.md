---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

﻿# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概览

自部署博客系统，由两个**独立的 Bun 包**组成（没有 workspace 根包，依赖要分别安装）：

- `ohmyblog-backend/` — Bun + Elysia + Drizzle + SQLite
- `ohmyblog-frontend/` — Vue 3 + Vite + Tailwind 4 + Pinia

生产形态是**单个可执行文件**：前端产物被塞进后端的 `public/`，由 Elysia 一起托管。所以前后端不是两个可独立部署的服务，改动时要考虑同源假设。

## 常用命令

后端（在 `ohmyblog-backend/` 下执行 —— 数据目录和迁移目录都基于 `process.cwd()`，换目录会导致读错路径）：

```bash
bun run dev            # 热重载，:3000；OpenAPI 文档在 /openapi（仅开发环境）
bun run lint           # biome lint
bun run lint:fix       # biome check --write（含格式化 + import 排序）
bun run db:gen         # 改完 db/table/*.ts 后生成迁移 SQL（启动时自动 migrate，无需手动跑）
bun run db:studio      # Drizzle Studio
bun run reset-password # 带外重置密码：忘记密码又没配 SMTP 时的唯一自救途径
                       # 可选 --disable-2fa 一起关掉两步验证（验证器丢了时用）
                       # 实现是 src/index.ts 的子命令而非 scripts/ 脚本，
                       # 因为 build.ts 只编译 src/index.ts，scripts/ 进不了二进制。
                       # 部署后对应 ./ohmyblog reset-password，
                       # Docker 里 docker exec -it -u 10001 <容器> /app/ohmyblog reset-password
                       # （-u 10001 不能省：以 root 跑会让 SQLite 的 -wal/-shm 归 root，主程序随后写不动）
bun run email          # react-email 预览服务，调 src/templates/*.tsx 用
bun run build:linux    # 单文件编译，产物在 scripts/dist/<platform>/
bun run docker         # 多阶段镜像（build context 是仓库根）
```

前端（在 `ohmyblog-frontend/` 下执行）：

```bash
bun run dev            # Vite :5173，/api、/feed、/sitemap.xml、/robots.txt 代理到 :3000
bun run type-check     # vue-tsc --build；会同时检查 Eden 引入的后端类型链
bun run build          # type-check + build-only 并行；CI 和 Docker 使用这个完整门禁
bun run build-only     # 仅打包，不做类型检查；只适合单独排查 Vite 构建
bun run format         # prettier --write .
```

**没有任何测试。** 后端的 `test` 脚本是 `exit 1` 占位，前端没有测试脚本，仓库里也没有 `*.test.*` / `*.spec.*`。不要去找测试或假装能跑测试；验证靠 `type-check` + `lint` + 手跑。

### type-check 的跨端约定

`vue-tsc` 会顺着 `@server/app` 检查 Eden Treaty 的完整后端类型链，这是有意的跨端契约门禁，不是噪音。`tsconfig.app.json` 已接入后端的 `bun-types`，并为 React Email 模板使用 React JSX 环境；当前基线是 **零报错**。任何 `../ohmyblog-backend/**` 报错都要按真实类型错误处理，不要过滤路径、跳过检查或改回 `build-only`。

因为类型链会解析后端依赖，运行前端 `type-check` / `build` 前必须先在 `ohmyblog-backend/` 执行过 `bun install`。

## 架构要点

### 端到端类型共享（最容易踩的地方）

前端通过 Eden Treaty 直接复用后端类型，`src/api/client.ts` 里 `treaty<ServerApp>(window.location.origin)`。这依赖一组跨包路径别名，**必须同时写在两处**，只改一处会出现「Vite 能跑但 type-check 挂」或反之：

- `ohmyblog-frontend/vite.config.ts` 的 `resolve.alias`
- `ohmyblog-frontend/tsconfig.app.json` 的 `compilerOptions.paths`

别名包括 `@server/app`、`@server/dtos/*`、`@server/db/constants/*`、`@server/db/table/*`，以及把 `elysia` 和 `@sinclair/typebox` 强制指向**后端的 node_modules**（两边版本必须一致，否则类型推导直接崩）。推论：**前端的 type-check / build 需要后端已经 `bun install`**。

`db/constants/*.ts` 是前后端共享的字面量 SSOT，必须保持零依赖（纯 `as const` 数组 + 派生类型），前端才能安全 import。前端统一从 `src/api/shared.ts` 再导出，组件不要直接写 `@server/...`。

新增后端路由后，记得在 `src/index.ts` 的 `.group("/api", ...)` 里挂上，否则 Eden 的类型树里根本没有这个端点，前端调用会报「属性不存在」。

### 后端分层

`routes/*.route.ts` → `services/*.service.ts` → `daos/*.dao.ts` → `db/`（Drizzle）。

- route 只做参数校验（DTO）、鉴权（`beforeHandle`）和组装返回值，不写业务逻辑
- service 抛 `BusinessError`（`plugins/errors.ts`）表达可预期失败，带 `status`；默认 `silent: true` 不写 error 日志
- dao 只关心 SQL；缓存逻辑抽到 `daos/caches/*.cache.ts`，不要污染 dao
- service/dao 都是 `class Xxx {}` + 文件底部 `export const xxxService = new XxxService()` 单例
- DTO 用 Elysia 的 TypeBox（`t.Object`），末尾统一 `export type TXxxDTO = Static<typeof XxxDTO>`；枚举用 `utils/typebox.ts` 的 `tStringEnum(常量数组)` 生成，保持和 `db/constants` 同源

### 统一响应封装

`plugins/response.plugin.ts` 全局包装：成功 → `{ success: true, data: <handler 返回值> }`，失败 → `{ success: false, data: { message, field? } }`。已经是 `Response` / `Blob` / 含 `success` 字段的对象会原样放行。

前端 `src/api/client.ts` 的 `unwrap()` 负责拆封，**失败时 throw 的是 `data` 里的内容**——通常是后端那句中文 message 字符串，不是 Error 对象。所以前端 catch 里会看到 `if (error === "配置不存在")` 这种字符串比较，这是设计如此，不是 bug。

### 鉴权与初始化引导

- `plugins/auth.plugin.ts`：JWT 存 cookie `auth_token`，全局 `derive` 把 `user` 注入 context（无 token 则为 `null`，不抛错）；`.macro` 提供 `role: "admin"` 写法
- `plugins/adminGuard.ts` 的 `ensureAdminIfExists`：**系统还没有任何 admin 时放行**，用于 setup 向导阶段能调用管理接口。绝大多数管理端路由用的是它而不是硬性 `role`
- 前端 `router/index.ts` 全局守卫先查 `/api/health` 的 `initialized`（即「是否已存在 admin」）：为 false 时一切重定向到 `/setup`，为 true 时禁止再进 `/setup`

`biome.json` 里 `noNonNullAssertion` 被刻意关掉：`beforeHandle` 保证了 handler 里 `user!` 是安全的，Biome 静态分析看不出来（详见后端 README）。

### 演示模式（DEMO_MODE）

`DEMO_MODE` 是与 `NODE_ENV` **正交**的布尔开关（演示站本身也是 `production` 部署，只是额外禁写），不要把它做成第三个 `NODE_ENV` 值——`isProduction()` 控制着 cookie 的 `secure`/`sameSite`、SQL 日志、OpenAPI 挂载，改动 `NODE_ENV` 的取值域会连带影响这些。

- `utils/demo.ts`：`isDemoActive()` = 开关打开 **且** 已有 admin（语义对齐 `ensureAdminIfExists`，系统未初始化时演示限制整个不生效，setup 向导照常可用）。`auth.plugin.ts` 的 derive 在无有效 token 时据此注入虚拟管理员 `__demo__`，于是所有 `ensureAdminIfExists` 路由对游客可读，**route 文件一行都不用改**
- `plugins/demo.plugin.ts`：全局 `onBeforeHandle` 按 **HTTP 方法** 拦截写操作，不是维护写接口清单。**新增写路由不需要来这里登记，默认就被挡住**；反过来说，要放行必须显式加进 `DEMO_ALLOWED_PATHS`
- 虚拟身份只存在于单次请求的 context，不签 JWT、不下发 cookie，没有可窃取或伪造的凭证
- `config.route.ts` 里 `isAdmin` 额外排除了演示身份，避免游客读到 `isPublic: false` 的 smtp 配置（含密码）
- `/api/health` 返回的 `demo` 是**生效值**（开关 && 已初始化），前端直接拿它决定要不要显示演示横幅


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JGG0sbp66/ohmyblog](https://github.com/JGG0sbp66/ohmyblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
