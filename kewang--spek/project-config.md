---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

spek 是一個 OpenSpec 內容檢視器，提供四種使用方式：
1. **Web 版**：本地唯讀 Web 應用（Express + React SPA），使用者啟動後在 UI 中選擇 repo 路徑瀏覽
2. **VS Code Extension**：直接在 VS Code 中開啟 Webview Panel 瀏覽當前 workspace 的 OpenSpec 內容
3. **IntelliJ Plugin**：在 IntelliJ IDEA 系列 IDE 中透過 Tool Window + JCEF 瀏覽 OpenSpec 內容
4. **Demo**：獨立靜態 HTML（`docs/demo.html`），內嵌 spek 自身的 openspec 資料，可部署至 GitHub Pages

## Tech Stack

- **Core**: `@spek/core` — 共用邏輯（scanner、tasks parser、型別定義），純 Node.js
- **Frontend**: React 19 + Vite + TypeScript + Tailwind CSS v4
- **Backend**: Express.js (Node.js) — 讀取本地檔案系統提供 REST API
- **VS Code Extension**: Webview Panel + esbuild bundling
- **IntelliJ Plugin**: Kotlin + JCEF + IntelliJ Built-in Server
- **Markdown**: react-markdown + remark-gfm（含 BDD 語法高亮）
- **Search**: Server-side 全文搜尋 + Fuse.js
- **Routing**: React Router v7（Web: BrowserRouter, Webview: MemoryRouter）

## Project Structure (Monorepo)

```
packages/
├── core/       # @spek/core — 純邏輯，無框架依賴
│   └── src/    # scanner.ts, tasks.ts, git-cache.ts, types.ts
├── web/        # @spek/web — Express + React 應用
│   ├── server/ # Express API server
│   └── src/    # React SPA + API adapters
├── vscode/     # spek-vscode — VS Code Extension
│   ├── src/    # extension.ts, panel.ts, handler.ts
│   └── webview/ # Vite build output（由 web build:webview 產出）
└── intellij/   # spek-intellij — IntelliJ Platform Plugin
    ├── src/main/kotlin/com/spek/intellij/  # Kotlin 原始碼
    └── src/main/resources/webview/          # Vite build output（由 web build:intellij 產出）
scripts/        # Build 工具（build-demo.ts）
docs/           # 靜態產出（demo.html，GitHub Pages 部署）
.agents/
└── skills/     # Claude Code skills（原始檔）
    └── frontend-design/  # 前端設計指引 skill
.claude/
└── skills/     # Symlinks → .agents/skills/（Claude Code 自動偵測）
```

## Development Commands

```bash
npm install              # 安裝所有 workspace 依賴
npm run dev              # 啟動 Web 版：Vite (5173) + Express (3001)
npm run build            # Build core + web
npm run build:core       # Build @spek/core
npm run build:web        # Build @spek/web（web 版 production build）
npm run build:webview    # Build webview assets（給 VS Code extension 用）
npm run build:vscode     # Build VS Code extension
npm run build:demo       # Build 獨立 demo HTML（docs/demo.html）
npm run build:intellij   # Build IntelliJ webview assets
npm run type-check       # TypeScript type check
npm run test -w @spek/core  # 跑 @spek/core 的 unit test（node:test + tsx）
```

**Web 開發**：`npm run dev` 後存取 http://localhost:5173

**VS Code Extension 打包**：
```bash
npm run build -w @spek/core && npm run build:webview -w @spek/web && npm run build -w spek-vscode
cd packages/vscode && npx vsce package --no-dependencies
```

**IntelliJ Plugin 打包**：
```bash
npm run build -w @spek/core && npm run build:intellij
cd packages/intellij && ./gradlew buildPlugin
# 產出: packages/intellij/build/distributions/spek-intellij-*.zip
```

## Architecture

### Core Module (`@spek/core`)
純函式 + 型別定義，可被 web server 和 extension host 共用：
- `scanOpenSpec(basePath)` — 掃描單一目錄的 OpenSpec 結構
- `scanOpenSpecAggregated(basePath, { aggregate })` — 跨 worktree 聚合掃描：探索同 repo 全部 worktree，active changes 聯集並附來源、archived 依 slug 去重、specs 取主 worktree；單一 worktree / 非 git / 關閉聚合時等同 `scanOpenSpec`
- `readSpec(basePath, topic)` — 讀取單一 spec（含歷史）
- `readChange(basePath, slug)` — 讀取單一 change
- `readSpecAtChange(basePath, topic, slug)` — 讀取特定 change 中的 spec 歷史版本
- `buildGraphData(basePath)` — 建立 spec-change 關聯圖資料
- `buildGraphDataAggregated(basePath, { aggregate })` — 跨 worktree 聚合的關聯圖（change 節點 id 以 `change:<worktreeKey>:<slug>` 命名避免碰撞）
- `listWorktrees(basePath)` — 以 `git worktree list --porcelain` 列出同 repo 全部 worktree；非 git / 無 `git` 時回 `[]`
- `shouldUsePolling(path, opts?)` / `pollingInterval(env?)` — 判定檔案監看是否該改用 polling（`watch-polling.ts`）。原生事件（inotify）在 9p/drvfs/NFS/CIFS 等掛載上不傳遞（devcontainer/WSL），故依「被監看路徑的 fstype」決定：純函式 `decidePolling` 套用優先序「明確覆寫（`SPEK_WATCH_POLLING`/`CHOKIDAR_USEPOLLING`）→ fstype 偵測（讀 `/proc/mounts`）→ remote 環境保底」。Web/VS Code 傳給 chokidar `usePolling`；IntelliJ 以 Kotlin 對齊版（`WatchPolling.kt`）在需要時改走輪詢掃描執行緒
- `parseTasks(content)` — 解析 tasks.md checkbox
- `extractHeadings(content)` / `slugifyHeading(text)` — 解析 markdown h2/h3 並產生穩定 slug，給 spec detail TOC 與 VS Code sidebar 共用（從 `@spek/core/headings` subpath 引入，避免 webview bundle 把 server-only 模組打包進去）
- 共用型別：`OverviewData`, `SpecInfo`, `ChangeInfo`, `ChangeDetail`, `GraphData`, `WorktreeInfo`, `WorktreeSource`, `Heading` 等

### API Adapter Pattern
前端透過 `ApiAdapter` 介面抽象通訊層：
- `FetchAdapter` — Web 版 + IntelliJ 版，呼叫 REST API（支援自訂 `baseUrl` 和 `dirParam`）
- `MessageAdapter` — VS Code Webview 版，透過 `postMessage` 與 extension host 通訊
- `StaticAdapter` — Demo 版，從 build time 內嵌的 `window.__DEMO_DATA__` 讀取靜態資料
- 透過 `ApiAdapterContext` (React Context) 注入

### API endpoints（Web 版，所有 openspec routes 接受 `dir` query param）

`/changes`、`/overview`、`/graph`、`/watch` 另接受 `aggregate` query param（預設 true，跨 worktree 聚合；`aggregate=false` 關閉）。`/changes/:slug` 接受 `wt`（worktree key）以辨識同名 slug 的來源 worktree。

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kewang/spek](https://github.com/kewang/spek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
