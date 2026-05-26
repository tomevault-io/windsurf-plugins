---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. You should response in Chinese.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. You should response in Chinese.

## Project Overview

mobile-gym is a **simulated Mobile environment (Android-like)** built with React + Vite + TypeScript + Tailwind CSS v4. It serves as a training and benchmarking platform for mobile GUI Agents. The simulator runs in a browser and exposes JavaScript APIs (`__SIM__`, `__OS__`, `__SIM_INPUT__`, `__SIM_QUERY__`, `__SIM_TIME__`, `__SIM_LOCATION__`, `__SIM_FS__`) for **task management, trajectory data synthesis, and benchmark orchestration**. The Agent only sees screenshots.

User-facing documentation under `docs/` and `bench_env/docs/` is in **English**. App code, UI labels, and inline comments inside `apps/` / `system/` follow existing project conventions — that's mostly **Chinese** for the consumer-app modules (WeChat, Alipay, etc.) and English for the OS layer and benchmark framework.

### 类型检查策略

- **小修改**（改几个文件、改样式、加数据等）— **不需要**跑 `tsc --noEmit`，依赖 IDE 实时检查即可
- **大修改**— 完成后跑一次 `npx tsc --noEmit` 确认无类型错误

### ESLint

```bash
npm run lint          # 检查 os/ 和 apps/ 下的运行时代码
```

当前规则：禁止裸 `Date.now()` 和任何形式的 `new Date(...)`（含带参形式，必须通过 `TimeService`）。配置见 `eslint.config.js`。

### Navigation Artifact Generation (run after modifying navigation declarations)

```bash
# One-shot: consistency check + schema nav graph + action tasks
node scripts/build_nav_artifacts.mjs <AppName>

# With data graph generation
node scripts/build_nav_artifacts.mjs <AppName> --data data/index.ts

# Skip tasks, only update graphs
node scripts/build_nav_artifacts.mjs <AppName> --skip-tasks
```

### Consistency Checking

```bash
node scripts/check_navigation_declaration_consistency.mjs <AppName> --actions
```

### Benchmark Environment (Python)

如果要运行 python，优先使用 conda 环境，本机理应安装过。

```bash
pip install playwright aiohttp
playwright install chromium

python -m bench_env.run --list                    # List all tasks
python -m bench_env.run --list --suite wechat      # Filter by suite
python -m bench_env.run --task-id <id> --env-url http://localhost:3000 --agent <type>
```

Supported agent types: `autoglm`, `gelab`, `generic`, `generic_v2`, `human`, `venus`, `gui_owl`, `uitars`.

## Architecture

The project has three main layers plus dev tooling. It is a single Vite project (not a monorepo). Path alias: `@/*` maps to the project root.

### OS Layer (`os/`)

The simulated Android system:

- **`OSContext.tsx`** — Thin React Context Provider; delegates to TaskManager, BackDispatcher, IntentResolver; exposes `window.__OS__` and `window.__SIM__` global APIs
- **`TaskManager.ts`** — Task/Activity栈管理（volatile，刷新=重启；状态可通过 `__SIM__.getState()`读取）。每个Task有 `stack: ActivityInstance[]`支持多Activity。`finishActivity()`：stack>1时弹顶部Activity；stack=1时**永不销毁Task**——有 `launchedByTaskId`就激活caller并consume，否则 `goHome()`，Task仍留在Recents（销毁需显式 `__OS__.closeApp` / Recents swipe）。`wasExternallyRouted`标志当前仅影响 `LAUNCH_APP`从桌面重激活时是否清除 `launchedByTaskId`（详见 `docs/platform/os/task-lifecycle.md`）
- **`BackDispatcher.ts`** — Priority-based back key handler. Components register with priority (e.g., PermissionDialog:1000, Shade:800, Keyboard:700, App:100). Includes frame-level deduplication to prevent double-back when edge-swipe gesture and backdrop click fire in the same frame
- **`IntentResolver.ts`** — Intent matching, chooser state management, startActivityForResult
- **`AppNavigatorRegistry.ts`** — Event-driven app/activity navigator registration. Uses CustomEvent + Promise pattern (replaces polling). Navigator `navigate(path, options?)` accepts optional `{ replace?: boolean }` — OS uses this to control push (existing tasks) vs replace (new tasks) when routing via `openApp`
- **`SystemShell.tsx`** — Desktop, status bar, gesture handling, app rendering container. Apps stay mounted when backgrounded (hidden via `display:none`), preserving React state. Implements **adjustResize**: wraps each Activity in a `data-adjust-resize` div that shrinks by keyboard height when keyboard is visible, so App flex layouts auto-adapt. When keyboard is active, the container gets `data-keyboard-active` attribute — elements with `data-hide-on-keyboard` are automatically hidden via global CSS
- **`AppStateRegistry.ts`** — Dual-layer state: runtime registry (from mounted apps) + persistent readers (localStorage fallback). External access via `getAllAppStates()`
- **`types.ts`** — Core type definitions (`AppId = string`). `AppId` is a plain string alias — apps are auto-discovered, no manual type union needed
- **`types/manifest.ts`** — `AppManifest` type definition (id, packageName, displayName, displayNameEn, aliases, version, icon, theme, etc.)
- **`data/appRegistry.tsx`** — App registry: auto-discovers manifests (`apps/*/manifest.ts`, `system/*/manifest.ts`) and entry components (`apps/*/*App.tsx`, `system/*/*App.tsx`) via `import.meta.glob`. **New apps do NOT need to register here**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Purewhiter/mobilegym](https://github.com/Purewhiter/mobilegym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
