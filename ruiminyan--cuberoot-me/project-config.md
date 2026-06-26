---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库布局

仓库 `RuiminYan/cuberoot.me`(自定义域名 `cuberoot.me`),同时托管：

1. **根目录** —— static.cuberoot.me 服的共享静态(`tools/` forks + `stats/` WCA JSON)+ 顶层 `solver/`/`fmc/`(Rust)+ 仓库基建(`ops/` `docs/` workflows)。早期 Vite build 残留 + GH Pages 镜像已于 2026-06-14 全部清除(GH Pages 站已禁用,DNS 本就不走它)。
2. **`core/`** — pnpm + Turbo monorepo，所有新开发都在这里：
   - `packages/client` — **React 19 + Next.js 16 (App Router, Turbopack)** ← **唯一前端工作区** (Phase 4 2026-05-27 切完;退役的 Vite `packages/client` + Capacitor 移动壳已于 2026-06-14 整包移除)
   - `packages/server` — Hono + **PostgreSQL 13**（WCA OAuth + recon + alg 公式库 + 训练数据，部署到云服务器;2026-05-06 从 MariaDB 迁过来,MariaDB 服务 + 数据已完整卸载)
   - `packages/shared` — 共享类型(`shared/src/alg.ts` 等);**公式数据全部在 PG `alg_sets/alg_cases` 两张表** (2026-05-06 从 JSON 迁过来),`loadAlg(puzzle, set)` 走 `/api/alg/sets/:p/:s` fetch
   - `packages/visualcube` — 自有 visualcube 封装;CI/server bundle 前必须先 build (`pnpm -F @cuberoot/visualcube build`,产 `dist/index.js`),否则 esbuild/Vercel build 找不到 export
   - `packages/stats-build` — WCA 统计生成管道（独立 CI 日更，stats.yml `cron 0 20 * * *`，跟 WCA dump 上游天更）
3. **`solver/`** — 顶层(pnpm workspace 外,非 package)。魔方求解引擎(Rust,2026-05-31 从已退役的 cube-solver-rust 导入,monorepo 为唯一源)。产 native 分析器喂 `/scramble/*` 数据管道(`update_cross_stats.ps1` 的 `$SolverDir` 指这)+ 编 WASM 给浏览器端。`target/ tables/(~34GB) pkg-web/ pkg-node/` 本地 gitignored(只本机有,repo/CI/线上都没有)。

## 12 个模块的归属（重要）

首页(`app/[lang]/page.tsx` 渲染 `components/LandingPage`)列出多入口。部分 fork 不能改:

| 模块 | 路由 | 位置 | 来源 | 可改? |
|------|------|------|------|-------|
| Solver | `/solver` | 根目录静态 HTML(只本机 nginx serve,Vercel 上走 `tools/[...slug]` 反代 static.cuberoot.me) | fork of [or18/RubiksSolverDemo](https://github.com/or18/RubiksSolverDemo) | ❌ upstream |
| Alg Trainer | `/alg-trainers` | 根目录静态 HTML(同上) | fork of [mihlefeld/Alg-Trainers](https://github.com/mihlefeld/Alg-Trainers) | ❌ upstream |
| csTimer | `/cstimer` | iframe → `/tools/cstimer/`(同上 fallback) | integrated from [cs0x7f/cstimer](https://github.com/cs0x7f/cstimer) | ❌ upstream |
| WCA Stats（数据管道） | `/wca` | `core/packages/stats-build` | 基于 [jonatanklosko/wca_statistics](https://github.com/jonatanklosko/wca_statistics) 的 TS 重写 | ⚠️ 管道已重写，UI 自有 |
| Score Calculator (HTH) | `/calc` | `core/packages/client/app/[lang]/calc/` | ported from [carykh/hthgrapher](https://github.com/carykh/hthgrapher) | ✅ |
| 1v1 Battle | `/battle` | `core/packages/client/app/[lang]/battle/` | ported from [MatteoColombo/cube_challenge_timer](https://github.com/MatteoColombo/cube_challenge_timer) | ✅ |
| Recon | `/recon` | `core/packages/client/app/[lang]/recon/` | 自有 | ✅ |
| Trainer（公式计时训练，全 41 套） | `/trainer` | `core/packages/client/app/[lang]/trainer/` | 自有 | ✅ |
| Recognize（PLL 识别训练，看图答字母） | `/recognize/pll` | `core/packages/client/app/[lang]/recognize/[algSetId]/` | 自有 | ✅ |
| Frame Count | `/frame-count` | `core/packages/client/app/[lang]/frame-count/` | 自有（WebCodecs + mp4box.js） | ✅ |
| Distribution | `/wca/viz` | `core/packages/client/app/[lang]/wca/viz/` | 自有 | ✅ |
| Comp (比赛中心:搜索/日历/地球/实时成绩) | `/wca/comp` | `core/packages/client/app/[lang]/wca/comp/` | 自有 | ✅ |
| Scramble（打乱难度 / 长度分布） | `/scramble/stats` | `core/packages/client/app/[lang]/scramble/stats/` + 数据 `stats/scramble/*.json`（长度走 CI 日更 `build_scramble_lengths.ts`） | 自有 | ✅ |
| Mosaic（魔方马赛克生成） | `/mosaic` | `core/packages/client/app/[lang]/mosaic/` | ported from [Roman-/mosaic](https://github.com/Roman-/mosaic) | ✅ |
| Blog | `blog.cuberoot.me`(`/blog` redirect 过去) | 独立 repo `RuiminYan/cuberoot-blog` | 外部托管 | — |

改 upstream 模块前先问用户；要改就只改 fork 后新增/包装的部分。**前端只有 client 一个工作区**。

## 部署拓扑 (Phase 4 后 — 2026-05-27)

> **push = 上线(默认别 push)**:`git push origin main` 会让 Vercel + 服务器**立刻自动重建上线**;`git commit` 只是本地(免费、不部署、多 AI 共仓也能互见)。**默认只 commit + 本地验证(dev/typecheck/test/Playwright),不 push**。仅这些情形才 push,且**先告知用户**:① 用户明说上线 / push;② DB 迁移要在线上库生效(本地能测,线上只在服务器部署重启时自动 apply);③ 改 nginx / systemd / 服务器 env / 后端 API 行为(仅生产存在,本地无等价);④ bug 只在生产复现需部署验证;⑤ 线上正坏的紧急修。普通功能 / bug / 重构 / UI / 文案做完 commit 即停,攒着等用户说上线。

- **主域 `cuberoot.me` / `www.cuberoot.me`** 走 **DNS 分线路** (provider 自带分流):
  - 一条线路 → 自有服务器 IP → nginx `proxy_pass 127.0.0.1:3002` → systemd `cuberoot-next` (Next standalone)。vhost `ops/nginx/www.cuberoot.me.conf`,改 nginx 走 `deploy_nginx.yml`(scp + `nginx -t` + reload + 失败回滚 .bak)。
  - 另一条线路 → Vercel Hobby `cuberoot-me` project → 同一份 Next 代码 + Vercel edge。Vercel 自动从 GitHub main 跑 build,部署是 push-triggered。
- **`static.cuberoot.me`** — 自有服务器 nginx 独立 vhost,只服 `/www/wwwroot/toolkit/{tools,stats}/`(forks 静态资源 + WCA stats JSON),CORS:* 给 Vercel function fallback。2026-05-27 替代退役的 `vite.cuberoot.me`。
- **`next.cuberoot.me`** — 同一套 systemd `cuberoot-next` 反代 :3002,作 staging 子域 / 别名。
- **systemd Next standalone 部署**:`deploy_next.yml`(push `core/packages/{client,shared,visualcube}/**` 触发) CI build → tar `.next/standalone/`(自带 node_modules) → scp → 服务器原子换 `/www/wwwroot/toolkit-next/` + 健康检查 :3002,挂了自动回滚 .bak。`start.sh` 包装定位 standalone entry,systemd unit 在 `ops/systemd/cuberoot-next.service`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuiminYan/cuberoot.me](https://github.com/RuiminYan/cuberoot.me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
