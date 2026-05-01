---
trigger: always_on
description: Epherome is a cross-platform Minecraft launcher built with **Tauri 2** (Rust backend) +
---

# Epherome — Agent Instructions

Epherome is a cross-platform Minecraft launcher built with **Tauri 2** (Rust backend) +
**React 19 + TypeScript** (frontend) + **Tailwind CSS v4**. Vite bundles the frontend;
Cargo compiles the native desktop shell.

---

## 1. Project Overview

| Layer | Technology | Role |
|---|---|---|
| Frontend | React 19, TypeScript 5.9, Vite 7 | UI, state, launch orchestration |
| Styling | Tailwind CSS v4 (via `@tailwindcss/vite`) | Utility-first, no component library |
| Icons | lucide-react | SVG icon set |
| IDs | nanoid | Unique IDs for accounts, instances, tasks |
| Backend | Rust, Tauri 2 | Native OS, file I/O, HTTP, process spawn |
| HTTP client | reqwest 0.11 (Rust) | All network requests go through Rust |
| Linter/Formatter | Biome 2 | Single tool replacing ESLint + Prettier |

Core business flows: Microsoft OAuth → Xbox/XSTS token chain → Minecraft token →
profile fetch; version JSON parsing; library/asset integrity checking; parallel
downloads; Fabric mod loader installation; Java detection and management.

---

## 2. Architecture & Directory Structure

```
src/
  main.tsx          # Entry: app init, data bootstrap, global error listener
  App.tsx           # Root: AppContext provider, sidebar nav, dialog overlay
  index.css         # Tailwind import + dark-mode custom variant
  components/       # 12 stateless UI primitives (Button, Input, Dialog, …)
  views/            # 11 full-page view components (*View.tsx)
  core/             # All business logic (launch, auth, download, assets, …)
    index.ts        # launchMinecraft orchestration entry point
    auth.ts         # Microsoft OAuth token chain
    download.ts     # File download, installMinecraft, installFabric
    assets.ts       # Asset index check/download
    libraries.ts    # Classpath/library resolution
    arguments.ts    # Minecraft launch argument parsing
    rules.ts        # OS platform rule compliance
    java.ts         # Java detection and version query
    parallel.ts     # ParallelManager (concurrent download class)
    skin.ts         # Skin fetching helper
  store/
    index.ts        # AppContext type + createContext + module-level globals
    data.ts         # UserData types, readUserData, writeUserData, fallbackUserData
    theme.ts        # updateTheme (data-theme attribute management)
  utils/
    fs.ts           # Typed wrappers over Rust FS commands (invoke calls only here)
    http.ts         # Typed wrappers over Rust fetch command (invoke calls only here)

src-tauri/src/
  lib.rs            # Plugin registration + invoke_handler command table
  main.rs           # Binary entry point
  core/
    auth.rs         # get_microsoft_auth_code (opens OAuth WebviewWindow)
    java.rs         # get_java_version (spawns java -version)
    runner.rs       # launch_minecraft (spawns process, emits process-output events)
    mod.rs
  utils/
    fs.rs           # read_text_file, write_text_file, exists, mkdir, read_dir, read_file, write_file
    http.rs         # fetch (reqwest-based, supports text/bytes response types)
    mod.rs
```

---

## 3. Commands

### Frontend

```bash
npm run dev          # Vite dev server only (no Tauri shell)
npm run build        # tsc type-check → vite build (full production build)
npm run preview      # Preview the production bundle
npm run lint         # Biome check --write (auto-fixes style + organizes imports)
npm run tauri dev    # Full Tauri app in development mode
npm run tauri build  # Production desktop binaries
```

### Verification After Changes

To verify frontend changes, run these two commands instead of `npm run build`
(which includes a slow Vite bundling step that is unnecessary for validation):

```bash
npm run lint         # Biome: auto-fix formatting + imports
npx tsc --noEmit    # TypeScript type-check only (no Vite build)
```

For Rust changes, run from the `src-tauri/` directory:

```bash
cargo clippy         # Lint
cargo fmt            # Format
```

### Rust Backend

```bash
# Run from src-tauri/ or use --manifest-path from root
cargo build
cargo clippy         # Lint
cargo fmt            # Format
```

### Tests

**There are no tests.** Do not add a test runner without explicit instruction.

---

## 4. Code Conventions

### 4.1 Naming Rules

| Entity | Convention | Examples |
|---|---|---|
| React components | PascalCase | `Button`, `DashboardView` |
| Component files | PascalCase `.tsx` | `Button.tsx`, `AccountsView.tsx` |
| View files | PascalCase + `View` suffix | `InstanceEditorView.tsx` |
| Non-component TS files | camelCase `.ts` | `auth.ts`, `download.ts`, `data.ts` |
| Functions | camelCase | `launchMinecraft`, `downloadFile` |
| Interfaces | PascalCase | `MinecraftClientJson`, `DialogOptions` |
| Type aliases | PascalCase | `ColorTheme`, `MinecraftVersionType` |
| Constants | camelCase | `fallbackUserData`, `errorList` |
| Local state variables | camelCase | `newJavaPath`, `modLoaderVersions` |
| Rust functions / commands | snake_case | `launch_minecraft`, `read_text_file` |
| Tauri IPC command strings | snake_case string literals | `"launch_minecraft"`, `"read_dir"` |
| Tauri event names | kebab-case strings | `"process-output"`, `"ms-auth-code"` |

### 4.2 TypeScript Type System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Epheromeers/Epherome](https://github.com/Epheromeers/Epherome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
