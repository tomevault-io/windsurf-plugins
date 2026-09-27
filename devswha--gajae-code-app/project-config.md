---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## What this is

Gajae Code App (`gajae-app`, v2.0.0-beta.x) — a self-hosted web + desktop UI for the GJC
coding agent. MIT. Four runtime layers:

- `src/` — React 19 SPA (Vite 7, Tailwind 4, react-router, i18next).
- `server/` — Express backend (`server/index.js` entry), SQLite via better-sqlite3,
  WebSocket, node-pty terminals. TypeScript + JS mixed, run through `tsx`.
- `native/gajae-core/` — Rust core, built to `dist-native/` by `scripts/build-rust-core.mjs`.
- `src-tauri/` — Tauri 2 desktop shell (Rust: `supervisor.rs`, `lifecycle.rs`,
  `navigation.rs`); packages the server as a payload and supervises it.

`shared/` is code shared between client and server (product identity, network hosts,
job projection protocol). `scripts/` holds build/release/verify tooling.

## Environment

- Node 22.22.2+ (22.x) or 24.15.0+ (24.x) — the test runner refuses other majors.
  On the primary Mac: `. "$HOME/.nvm/nvm.sh" && nvm use 22`.
- Rust/cargo required for `check:core`, `build:core*`, and the Tauri shell
  (`. "$HOME/.cargo/env"`).
- Bun **exactly 1.4.0** for `*.bun.test.ts` and `*.dom.bun.test.tsx` files (pinned in
  `scripts/fetch-bun.mjs`): `dist-native/bun` or PATH; fetch with
  `node scripts/fetch-bun.mjs`.
- `npm ci` applies the app-owned SDK lifecycle patch from
  `patches/gjc-sdk-lifecycle/manifest.json` through postinstall. Exact SDK/core/AI
  versions and complete before/after hashes are mandatory. Use
  `npm run apply:sdk-patch` / `npm run check:sdk-patch`; never hand-edit installed
  dependency files. Unknown local modifications must fail rather than be replaced.
- Server binds loopback by default (fail-closed; it can run shell commands).
  `SERVER_PORT` defaults to 3001, Vite dev on 5173. Do not export `SERVER_PORT=0`.
- A long-lived dev stack may already be running in tmux session `gajae-dev`
  (check `tmux ls` and `lsof -nP -iTCP:3001 -iTCP:5173 -sTCP:LISTEN`; its log is
  mirrored to `/tmp/gjc-dev/dev.log` and the address/operating notes live in
  `/tmp/gjc-dev/README.md`). Reuse it rather than starting a second
  `npm run dev` on the same ports. On the primary Mac it serves the tailnet:
  `HOST=$(tailscale ip -4) GAJAE_ALLOW_UNAUTH_REMOTE=1 npm run dev`. That
  override disables authentication on the bound address, so never combine it
  with a bind that is reachable outside the tailnet.
- Tauri builds choke on `CI=1`: use `env -u CI npm run tauri -- build`.
- A release-profile macOS build refuses to guess its updater mode: set
  `GJC_UPDATE_MODE=disabled` for ad-hoc/manual bundles, or the full production
  binding (`GJC_UPDATE_MODE=production`, `GJC_UPDATE_FEED_ORIGIN`,
  `GJC_UPDATE_PUBKEY`) for anything the updater will ship. See
  `scripts/release/MACOS-ACCEPTANCE.md`.
- **Desktop scope (owner decision, 2026-09-09): macOS (Apple Silicon) first.**
  The Linux desktop app is out of active development until the macOS app is
  complete. Do not plan, build, smoke, or gate work on Linux desktop packages;
  do not carry Linux desktop items forward as remaining work. The Linux
  *server* archive (self-host) stays in scope. Linux desktop packaging
  (`docs/DESKTOP-LINUX.md`, `desktop:build:linux`, the dispatch-only
  `desktop-linux.yml` lane) is kept, not maintained; touch it only when the
  owner asks for a Linux desktop build.

## Commands

```bash
npm run dev              # server (tsx, :3001) + vite client (:5173, loopback unless HOST is set); prebuilds rust core
npm test                 # all tests via scripts/run-tests.mjs (node:test + bun test)
npm run typecheck        # tsc on both tsconfig.json and server/tsconfig.json
npm run lint             # eslint src/ server/ shared/ scripts/ + configs
npm run check:core       # cargo fmt --check + clippy -D warnings + cargo test
npm run verify           # FULL GATE: audit + typecheck + check:core + test + test:e2e:gjc + lint + check:identity + build
npm run test:e2e:gjc     # 8 GJC wire/browser e2e tests (also part of verify; not part of npm test)
npm run desktop:dev      # Tauri dev shell
npm run server:payload:macos # embedded macOS server payload + sidecar (prerequisite for src-tauri cargo test)
GJC_UPDATE_MODE=disabled env -u CI npm run tauri -- build --bundles app # ad-hoc macOS app bundle (unsigned, no updater)
npm run server:payload:linux # Linux x64 self-host payload + pinned runtimes
# Linux desktop (out of scope; owner request only): env -u CI npm run desktop:build:linux,
# then npm run smoke:packaged-server -- --linux-root <extracted-dir> [--data-survival|--appimage-env]
```

Run a single test file (match the runner's env):

```bash
# server test (node:test via tsx)
TSX_TSCONFIG_PATH=server/tsconfig.json node --import tsx --test server/gjc-worker.test.ts
# client test
TSX_TSCONFIG_PATH=tsconfig.json node --import tsx --test src/stores/useSessionStore.test.ts
# bun-runtime test (files named *.bun.test.ts)
dist-native/bun test server/gjc-sdk-contract.bun.test.ts
# component test with a real DOM (files named *.dom.bun.test.tsx)
dist-native/bun test src/shared/view/ui/ActionMenu.dom.bun.test.tsx
```

`npm test` has a `pretest` that builds the Rust core (debug); tests fail without it.

Desktop shell CI is `.github/workflows/desktop-macos.yml` (PR/push-main on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devswha/gajae-code-app](https://github.com/devswha/gajae-code-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
