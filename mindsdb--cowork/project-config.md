---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md
# MindsHub Cowork — build & dev notes

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Electron 39 + Vite + React 19 + Tailwind desktop app with a FastAPI Python sidecar (`cowork-server`) managed via `uv`.

## Commands

### Build & run

```sh
# Full Electron build → release/mac-arm64/MindsHub Cowork.app
PATH="/opt/homebrew/opt/node@20/bin:$PATH" npm run pack
```

- Output: `release/mac-arm64/MindsHub Cowork.app`
- Confirm with: `stat -f "%Sm" -t "%H:%M:%S" "release/mac-arm64/MindsHub Cowork.app"`
- Code-sign warnings ("0 valid identities found") are expected in dev — ignore.
- Build is the only way to verify Python server changes; the renderer is bundled into the same artifact.

> **iCloud builds** — If the repo lives under `~/Documents` (iCloud Drive), codesign will fail with
> `resource fork, Finder information, or similar detritus not allowed` on the GPU Helper binary.
> The `scripts/strip-xattrs.js` `afterPack` hook clears `com.apple.provenance` and
> `com.apple.FinderInfo`, but iCloud re-tags binaries in the race window before signing.
> Build to `/tmp` instead and copy back:
>
> ```sh
> PATH="/opt/homebrew/opt/node@20/bin:$PATH" \
>   npx electron-builder --mac --arm64 --config.directories.output=/tmp/minds-build
> cp -R /tmp/minds-build/mac-arm64 release/
> ```

### Dev mode (renderer only, faster iteration)

```sh
PATH="/opt/homebrew/opt/node@20/bin:$PATH" npm run dev

# Web SPA (no Electron) — spins up cowork-server + Vite on http://localhost:5173/
npm run dev:web
npm run build:web   # → dist/renderer-web/
```

### Type checking

```sh
npm run typecheck   # main (tsconfig.main.json) + renderer (tsconfig.json) + tests (tsconfig.test.json)
```

### Testing

Vitest runs two projects — `main`
(node env, `src/main` + `src/shared`) and `renderer` (happy-dom env,
`src/renderer`). Tests are **colocated** (`foo.test.ts` beside `foo.ts`);
shared setup lives in `tests/` (autouse env scrub, `TZ=UTC`, network deny —
`fetch` throws unless a test installs its own mock).

```sh
npm test                     # unit + component (fast, no Electron/network)
npm run test:watch           # local TDD loop
npm run test:coverage        # + enforces coverage floors (vitest.config.ts)
npm run test:e2e             # Playwright Electron boot smoke (needs npm run build)
npm run check:cowork-purity  # no direct window.antontron outside platform/host.ts
```

**Norms:**

- **A PR that fixes a bug adds a regression test for it. A PR that adds logic
  adds tests for that logic.** Decision logic goes in pure functions
  (`src/main/update-logic.ts` pattern) tested directly; orchestration gets at
  most 1–2 integration-style tests with mocks.
- Coverage floors in [vitest.config.ts](vitest.config.ts) are **ratcheted from
  measured values — raise them when coverage grows, never lower them to make a
  failing PR pass**. `update-logic.ts` and `server-source.ts` are locked at
  100%.
- Never touch `window.antontron` inside `src/renderer/cowork/` — go through
  [src/renderer/platform/host.ts](src/renderer/platform/host.ts). CI enforces
  this.
- Test files never ship: production tsconfigs exclude `**/*.test.*`; keep it
  that way.
- CI: `tests-unit.yml` gates PRs (typecheck + purity + coverage + renderer
  build); `pack-smoke.yml` and `tests-e2e.yml` run nightly, non-blocking.
- The IPC channel map is snapshot-locked
  (`src/shared/__snapshots__/`) — renaming a channel is a **breaking protocol
  change** (OTA renderers can lag main), not a refactor.

### Python server

The backend is `cowork-server`, a separate package installed via `uv tool install`. It is **not** vendored in this repo.

```sh
# Dev: run from the sibling cowork-server/ source directory
uv run cowork-server

# Packaged: binary lives in ~/.local/bin/cowork-server (macOS/Linux)
# or %LOCALAPPDATA%/bin/cowork-server.exe (Windows)
```

FastAPI runs loopback-only at `127.0.0.1:26866`. CORS defaults to localhost origins only; override with `COWORK_ALLOWED_ORIGINS='["*"]'` for cloud/VPC deployments or an ingress-controlled environment.

**Bearer-token authentication** — on by default in local/desktop tenancy. Off by default in org mode, where `COWORK_REQUIRE_AUTH=true` isn't supported at all — the token would be mirrored into shared storage every org can read; org's own ingress is the auth boundary there. To pin a fixed token or opt out in local mode, set in `~/.cowork/.env`:

```
COWORK_REQUIRE_AUTH=false        # opt out (not recommended)
COWORK_AUTH_TOKEN=<your-token>   # omit to auto-generate on first startup
```

When `COWORK_AUTH_TOKEN` is empty, the server generates a cryptographically random token at startup and writes it back to `~/.cowork/.env`. The desktop app reads the same file and injects `Authorization: Bearer <token>` on every API request automatically. The `/api/v1/health/` endpoint is always exempt.

#### Install source & channel

Where cowork-server **and** its `anton-agent` dependency are installed from is centralized in [src/main/server-source.ts](src/main/server-source.ts) — shared by the installer and the auto-updater so they can't disagree (a PyPI updater must never clobber a git install, and vice-versa).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindsdb/cowork](https://github.com/mindsdb/cowork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
