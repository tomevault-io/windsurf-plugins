---
trigger: always_on
description: This file is for agentic coding tools. It's a map — read linked docs for depth.
---

# AGENTS.md

This file is for agentic coding tools. It's a map — read linked docs for depth.

## Repo overview

Nexu is a desktop-first OpenClaw platform. Users create AI bots, connect them to Slack, and the local controller generates OpenClaw config for the embedded runtime.

- Monorepo: pnpm workspaces
- `apps/controller` — Single-user local control plane for Nexu config, OpenClaw sync, and runtime orchestration
- `apps/desktop` — Electron desktop runtime shell and sidecar orchestrator
- `apps/web` — React + Ant Design + Vite
- `packages/slimclaw` — Repo-local Nexu-owned OpenClaw runtime contract, prepared runtime root, and staging/patch ownership for local dev and desktop packaging
- `packages/shared` — Shared Zod schemas
- `packages/dev-utils` — TS-first reusable utilities for local script tooling

## Project overview

Nexu is a desktop-first OpenClaw product. Users create AI bots via a dashboard and connect them to Slack. The system dynamically generates OpenClaw configuration and hot-loads it into the local runtime managed by the controller.

## Commands

All commands use pnpm. Target a single app with `pnpm --filter <package>`.

```bash
pnpm install                          # Install
pnpm --filter @nexu/shared build      # Build shared dist required by cold-start dev flows
pnpm dev start                        # Start the lightweight local stack: openclaw -> controller -> web -> desktop
pnpm dev start <service>              # Start one local-dev service: desktop|openclaw|controller|web
pnpm dev restart                      # Restart the lightweight local stack
pnpm dev stop                         # Stop the lightweight local stack in reverse order
pnpm dev stop <service>               # Stop one local-dev service
pnpm dev restart <service>            # Restart one local-dev service
pnpm dev status <service>             # Show status for one local-dev service
pnpm dev logs <service>               # Show active-session log tail (max 200 lines) for one local-dev service
pnpm dev inspect screenshot           # Capture the current desktop window screenshot (dev desktop only)
pnpm dev inspect eval "<expr>"        # Evaluate a JS expression in the desktop renderer (dev desktop only)
pnpm dev inspect dom                  # Dump the current desktop renderer DOM summary (dev desktop only)
pnpm dev inspect logs                 # Show buffered desktop renderer console/error logs (dev desktop only)
pnpm dev:controller                   # Legacy controller-only direct dev entrypoint
pnpm dist:mac                         # Build signed macOS desktop distributables
pnpm dist:mac:arm64                   # Build signed Apple Silicon macOS desktop distributables
pnpm dist:mac:x64                     # Build signed Intel macOS desktop distributables
pnpm dist:mac:unsigned                # Build unsigned macOS desktop distributables
pnpm dist:mac:unsigned:arm64          # Build unsigned Apple Silicon macOS desktop distributables
pnpm dist:mac:unsigned:x64            # Build unsigned Intel macOS desktop distributables
pnpm dist:win:local                   # Fast local Windows packaging check: reuse existing builds/runtime/sidecars when available and validate dir-only output
pnpm probe:slack prepare              # Launch Chrome Canary with the dedicated Slack probe profile
pnpm probe:slack run                  # Run the local Slack reply smoke probe against an authenticated DM
pnpm --filter @nexu/web dev           # Web only
pnpm build                            # Build all
pnpm check:esm-imports                # Scan built dist for extensionless relative ESM specifiers
pnpm typecheck                        # Typecheck all
pnpm lint                             # Biome lint
pnpm format                           # Biome format
pnpm test                             # Vitest
pnpm generate-types                   # OpenAPI spec → frontend SDK
```

After API route/schema changes: `pnpm generate-types` then `pnpm typecheck`.

This repo is desktop-first. Prefer the controller-first path and remove or ignore legacy API/gateway/container-era assets when encountered.

## Branch model

- `main` is the integration branch and should stay releasable.
- Do feature work on short-lived branches named with a clear prefix such as `feat/...`, `fix/...`, or `chore/...`.
- Prefer merging the latest `main` into long-running feature branches instead of rewriting shared history once a PR is under review.
- After a PR merges, sync local `main`, then delete the merged feature branch locally and remotely when it is no longer needed.

## Commit & PR conventions

- **No co-author trailer.** Never append `Co-Authored-By:` lines to commit messages.
- **Conventional commit prefix.** Use `chore:` for changes that are invisible to end users (CI/CD, issue bots, tooling, config). These are excluded from release notes. Use `feat:` / `fix:` / `docs:` etc. for user-visible changes.
- **Docs commit/PR prefix.** Documentation-only changes must use `docs:` for both commit titles and PR titles.
- **Non-user-facing commit/PR prefix.** Any change that is not user-facing and should not appear in release notes must use `chore:` for both commit titles and PR titles.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nexu-io/nexu](https://github.com/nexu-io/nexu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
