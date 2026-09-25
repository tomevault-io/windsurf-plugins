---
trigger: always_on
description: - Single-package, client-only plugin for the DSH (DeepSeek Harness) Web UI. It adapts the web UI on **touch-primary devices with a viewport below 1024px** (overlay drawer, full-width conversation, adapted settings/explorer/preview sheets, status-bar safe areas, composer row, stats line). The activation query is `MOBILE_QUERY = '(max-width: 1023px) and (pointer: coarse)'` (phone-chrome.ts): width alone cannot distinguish a phone from a narrow desktop window — split views and OS display scaling pu
---

# dsh-web-mobile

## Project

- Single-package, client-only plugin for the DSH (DeepSeek Harness) Web UI. It adapts the web UI on **touch-primary devices with a viewport below 1024px** (overlay drawer, full-width conversation, adapted settings/explorer/preview sheets, status-bar safe areas, composer row, stats line). The activation query is `MOBILE_QUERY = '(max-width: 1023px) and (pointer: coarse)'` (phone-chrome.ts): width alone cannot distinguish a phone from a narrow desktop window — split views and OS display scaling push a PC's CSS viewport below 1024px too (2026-08-30 PC leak). A mouse-driven window (`pointer: fine`) or pointer-less one stays desktop at **every** width; the desktop hide block in misc.css.ts is the exact complement of MOBILE_QUERY as a comma list and hides the slot-rendered controls outside the mobile branch. ONE deliberate exception (v2.4.1): the session-delete trio (menu item + confirm/error dialog) arms on `TOUCH_QUERY = '(pointer: coarse)'` at EVERY width, so a large tablet in landscape keeps the desktop layout but still gets the 「删除会话」 item.
- Names differ by boundary: README/GitHub project = `dsh-web-mobile`; npm package = `dsh-web-mobile`（2026-08-30 由 dsh-mobile-nav 改名而来，旧名连同 2.2.0/2.3.0 已整包 unpublish，npm 上不再存在）; patch row id = `dsh-web-mobile`（DOM 标记 `data-mobile-nav` 与 `?mobile-nav-debug=1` 参数刻意保留旧词根，见 Pitfalls）。
- 已被 [DSHA](https://github.com/qiannianhuanxiang/DSHA)（Android 启动器）内置为移动端适配（README 已致谢 @qiannianhuanxiang，commit ffb61b5）——DSHA 用户装 APK 即用。
- No monorepo, no application server, no workspace layer.
- Real entrypoints:
  - `cordis.patch.yml` inserts the single host plugin row.
  - `src/index.ts` is the host half: `apply()` makes the row visible to the host Loader, installs transparent gzip/brotli compression for large JSON responses (`src/compress.ts`), and registers the session-delete endpoint `/api/mobile-nav.session.delete` (work in `src/delete-session.ts`).
  - `package.json` exposes `./client` and declares `dsh.client.platform: "web"`; DSH discovers the browser half from `src/client/index.tsx`.
- Key layout（注释版仓库树；`(不入库)` = gitignore，外部 clone 不可见）:

  ```text
  dsh-web-mobile/
  ├─ src/                    ← 真源码，唯一该手改的地方
  │  ├─ index.ts             ← 宿主半区入口（apply 装响应压缩 + 会话删除端点）
  │  ├─ compress.ts          ← 进程级 prototype patch
  │  ├─ delete-session.ts    ← 会话删除纯核（DI、分代适配、可单测）
  │  └─ client/
  │     ├─ index.tsx         ← 浏览器半区入口（3 slots）
  │     ├─ debug.ts          ← ?mobile-nav-debug=1 诊断徽章
  │     ├─ components/       ← MobileNavToggle / MobileDrawerFooter / ComposerFileButton / open-files-panel.ts
  │     ├─ core/             ← reconciler-core.ts（零 import）+ raf-scheduler.ts · css-rules.ts · sessions-compat.ts · layout-compat.ts · icon-compat.ts（宿主图标跨代命名兼容）
  │     ├─ effects/          ← 17 个效果模块：phone-chrome · sidebar-swipe ·
  │     │                       gesture-guard · subagent-chip-touch · composer-keyboard-guard ·
  │     │                       composer-plus-toggle · workspace-chip-toggle · team-chip-toggle ·
  │     │                       model-menu-anchor ·
  │     │                       file-viewer-compat · aionui-compat · stats-line ·
  │     │                       preview-fullscreen ·
  │     │                       overlay-backdrop-fab · panel-exit · session-menu · session-row-fiber
  │     ├─ styles/           ← index.ts（base→layout→compat→misc 承载顺序）+ 4 个 .css.ts
  │     └─ i18n/locales.ts
  ├─ lib/                    ← 生成物：随 pnpm build 刷新，勿手改
  │  └─ types/…              ← d.ts+map；合并同 CSS 模块的 PR 在 .css.d.ts 冲突 → 重建
  ├─ scripts/
  │  ├─ build-client.mjs     ← 自研客户端打包器
  │  ├─ cdp-probe.mjs        ← 主探针 14 项核心断言（+6 集成，EXPECTED_FAILURES 基线）
  │  ├─ cdp-swipe-probe/failures · cdp-zoom-probe · cdp-compat-contracts (.mjs)
  │  ├─ css-structure-check.mjs ← CSS 结构检测器（已接入 test:core）
  │  └─ probes/              ← 22 个回归锚点（builtin-only，可单跑）
  ├─ tests/                  ← 31 个 .test.ts（node --test，type-stripping 直跑）
  ├─ docs/
  │  ├─ specs/               ← 8 篇权威设计文档（入库）
  │  ├─ audits/ · maintenance/pitfalls.md · upstream/（runbook + compat-contracts.json + host-jank-feedback.md）· fork-wzxmt-zhc/
  │  └─ debug/ · superpowers/ ← 本地不入库
  ├─ .github/workflows/ci.yml ← verify → test:core → build → git diff --exit-code lib
  ├─ assets/                 ← README 用图
  └─ .local-tests/ · .codegraph/ · .dsh-vision-toolkit/  ← 本地不入库（gitignore 噪音区）
  ```

## Commands

```sh
pnpm install                       # install (pnpm@11.7.0, lockfile v9)
pnpm verify                        # type-check host + client halves (tsc --noEmit)
pnpm test:core                     # node --test tests/*.test.ts (unit tests)
pnpm build                         # tsc host && tsc client && node scripts/build-client.mjs
npm run prepack                    # runs npm run build before packaging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mexiaosqwq/dsh-web-mobile](https://github.com/mexiaosqwq/dsh-web-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
