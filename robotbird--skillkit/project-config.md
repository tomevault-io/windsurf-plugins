---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


## What this is

Skillkit — manage "skills" across AI coding tools (Claude Code, Codex, Cursor, Trae, Workbuddy): browse the [skills.sh](https://www.skills.sh) marketplace, install/uninstall from market/GitHub/zip, and share installed skills via short links. A **pnpm-workspace monorepo** with two parts:

- **`apps/desktop`** — Electron desktop client (React 18 + TypeScript + Vite + better-sqlite3). This is what's open-sourced here.
- **`packages/types`** — `@skillkit/types`: cross-process shared types & constants (single source of truth).

The **server** (share short-link API + web 个人中心, Next.js 16) used to live here as `apps/server` but has **moved to a separate repo** at `../skillkit.net` (the product 官网 + backend). The desktop client talks to it over HTTPS at `https://skillkit.net` (share + auth endpoints; login = `https://skillkit.net/login`, no `account.` subdomain). `packages/types` is mirrored (vendored) into that repo so the wire contract stays in sync — **when you change share/auth types or the tool list here, update the vendored copy in `skillkit.net/lib/shared-types.ts` too**.

## Commands

Run from the repo root.

```bash
pnpm install                 # workspace install (links @skillkit/types, installs apps/desktop deps)

# Desktop (apps/desktop)
pnpm --filter desktop run rebuild   # REQUIRED: rebuilds better-sqlite3 against Electron's ABI. Run after install / Electron bumps.
                                   # ⚠️ Must use `run rebuild` — `pnpm rebuild` is a pnpm built-in that reruns install scripts against the *system* Node ABI (→ NODE_MODULE_VERSION mismatch); only the `scripts.rebuild` entry (`electron-rebuild -f -w better-sqlite3`) compiles against Electron's headers.
pnpm --filter desktop dev       # Full Electron app in dev (vite-plugin-electron launches Electron + watches all 3 bundles)
pnpm --filter desktop build     # tsc -p both tsconfigs (typecheck, noEmit) + vite build
pnpm --filter desktop pack      # build + electron-builder --dir (unpacked, for debugging)
pnpm --filter desktop dist      # build + electron-builder → release/ (mac dmg/zip, win nsis)

# Repo-wide (turbo)
pnpm dev        # turbo dev — runs desktop dev
pnpm build      # turbo build
pnpm typecheck  # turbo typecheck
```

There is **no test framework**. Typechecking: desktop via its `typecheck` script (`tsc -p` both tsconfigs).

## Architecture

### Monorepo layout

```
apps/desktop/   electron/ src/ shared/ build/ public/ vite.config.ts tsconfig*.json
packages/types/ src/index.ts            # @skillkit/types
pnpm-workspace.yaml  turbo.json  package.json (root: devDeps + turbo scripts only)
```

### `packages/types` (`@skillkit/types`)

Symbols consumed by the desktop (and, via a vendored mirror, by the server in `../skillkit.net`): `Tool`, `TOOL_LABELS`, `ALL_TOOLS`, `InstalledSkill`, `MarketSkill`, `InstallResult`, `InstalledFilter`, `Market*`, `ShareMeta`, `ShareCreateResult`, `ShareSourceInfo`, and share constants (`SHARE_BASE_URL`, `SHARE_TTL_MS`, `SHARE_MAX_BYTES`). **Single source of truth** — this is the canonical copy.

It is **pure TS with no build step**. The desktop bundler is taught to compile it:
- **desktop** (`apps/desktop/vite.config.ts`): `resolve.alias['@skillkit/types'] → ../../packages/types/src` in **all three** vite bundles (renderer / electron main / preload).

> The server repo (`../skillkit.net`) keeps a vendored copy at `lib/shared-types.ts` (exposed via a local `@skillkit/types` tsconfig path alias) — it is **not** a workspace consumer. Keep the two in sync on share / auth / tool-list changes.

### Desktop: three-process Electron model

`apps/desktop/electron/` (main, ESM, → `dist-electron/`), `apps/desktop/electron/preload.ts` (→ `preload.mjs`), `apps/desktop/src/` (React renderer, → `dist/`).

- **`shared/types.ts` is now a bridge layer** (NOT the source of truth): it re-exports the cross-end symbols from `@skillkit/types` **plus** defines desktop-only types (`UpdateAvailableInfo`, the `SkillkitApi` IPC contract, the `Window` global). Main process imports it as `../shared/types.js`; renderer as `@shared/types`. Keeping `shared/` inside `apps/desktop/` means **both paths stayed identical after the monorepo move** — no import rewrites were needed.
- **Adding a main-process capability still needs three coordinated edits**: an IPC handler in `electron/ipc.ts`, a method on `window.skillkit` in `electron/preload.ts`, **and** the matching signature in `shared/types.ts` (`SkillkitApi`).
- The `@shared` alias → `shared/` is wired in `apps/desktop/vite.config.ts` (renderer + main) and both `apps/desktop/tsconfig*.json`. Both use `moduleResolution: bundler`, so `@skillkit/types` resolves via the workspace symlink + its `exports` field — no tsconfig `paths` needed for it.
- **Renderer 有三个别名**：`@shared`(shared)、`@skillkit/types`(跨端类型)，外加 shadcn 约定的 `@/`(→ `apps/desktop/src`)，同样在 `vite.config.ts` + `tsconfig.json` 配好。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robotbird/skillkit](https://github.com/robotbird/skillkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
