---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Mandates
1. **Environment:** This project uses a Nix flake environment with `direnv`. Use `pnpm nx` for project tasks and `task` (Taskfile) for orchestrated workflows.
2. **Context Awareness:** Read `README.md` for domain-specific vocabulary (flow nodes, delta system) before starting complex tasks.
3. **File Editing:** Verify files exist before editing. Use `git status` and `git diff` to verify changes. **Never** revert changes you didn't author unless instructed. **Never** commit changes unless explicitly asked.
4. **Verification:** Always test, lint, and compile after making changes.

## Common Commands

### Build & Run
```bash
task dev:desktop                    # Full desktop app (Electron + React + Go Server)
pnpm nx run server:dev              # Go server with hot reload
pnpm nx run client:dev              # React frontend
pnpm nx run spec:build              # Regenerate from TypeSpec (after editing .tsp files)
pnpm nx run db:generate             # Regenerate sqlc (after editing .sql files)
cd apps/cli && task build:release   # Build CLI binary
```

### Testing
```bash
task test                           # All unit tests
pnpm nx run server:test             # Server tests only
pnpm nx run db:test                 # DB tests only
pnpm nx run cli:test                # CLI tests only

# Single Go test (use -run for specific functions)
cd packages/server && go test -run TestFunctionName ./path/to/package/ -v -timeout 30s
cd packages/db && go test -run TestFunctionName ./path/to/package/ -v -timeout 10s
cd apps/cli && go test -run TestFunctionName ./path/to/package/ -v -timeout 30s
```

### Lint & Fix
```bash
task lint                           # ESLint + format checks + golangci-lint
task fix                            # Prettier + Syncpack auto-fix
pnpm nx run server:lint             # Go linters (golangci-lint + norawsql + notxread)
```

### Benchmarks
```bash
task benchmark:run                  # Run benchmarks
task benchmark:baseline             # Save baseline
task benchmark:compare              # Compare against baseline
```

### Releasing
**Never** manually edit version numbers in `package.json`. Use Nx version plans:

```bash
# 1. Create a version plan (commits a .nx/version-plans/<name>.md file)
#    Bump types: patch, minor, major
#    Projects: desktop, cli, api-recorder-extension
task version-plan project=desktop   # Interactive — prompts for bump type + message

# Or create the file directly (non-interactive):
# .nx/version-plans/<descriptive-name>.md
# ---
# desktop: patch
# ---
# Changelog message here.

# 2. Commit & push the version plan file

# 3. Trigger the release workflow (via GitHub Actions):
gh workflow run release.yaml -f desktop=true   # -f cli=true, -f web=true, etc.
```

The release workflow reads version plans, bumps versions, creates git tags + GitHub releases,
and dispatches platform-specific build workflows (Electron Builder, Go binaries, etc.).

## Project Overview

DevTools is a local-first, open-source API testing platform (Postman alternative) — desktop app, CLI, and Chrome extension. Features request recording, visual flow building, and CI/CD integration.

## Architecture

### Monorepo Structure
- **`apps/desktop`** — Electron app (TypeScript/React, electron-vite)
- **`apps/cli`** — Go CLI (cobra). Embeds `packages/worker-js` (TypeScript worker bundled via tsup)
- **`packages/server`** — Go backend (Connect RPC, SQLite/LibSQL)
- **`packages/client`** — React frontend (TanStack Router/Query, Effect-TS, React Flow)
- **`packages/ui`** — Shared React component library (React Aria, Tailwind Variants, Storybook)
- **`packages/db`** — Go database package (`devtoolsdb`), sqlc generated code, SQLite drivers
- **`packages/spec`** — TypeSpec definitions → Protobuf → Go/TypeScript codegen (single source of truth)
- **`packages/worker-js`** — TypeScript worker bundled into CLI binary
- **`tools/`** — Custom Go linters (`norawsql`, `notxread`), benchmarking, spec emitter, ESLint config

### Go Workspace
`go.work` with Go 1.25. Modules: `apps/cli`, `packages/db`, `packages/server`, `packages/spec`, and tools.

### Naming Conventions
- **Services:** `s` prefix (`shttp`, `senv`, `sflow`, `suser`, `sworkspace`)
- **Models:** `m` prefix (`mhttp`, `mflow`, `menv`, `muser`)
- **RPC handlers:** `r` prefix (`rhttp`, `rflowv2`, `renv`)
- **IDs:** All use `idwrap.IDWrap` (ULID-based) from `packages/server/pkg/idwrap`

### Backend Layers (Server)
1. **RPC Layer** (`packages/server/internal/api`) — Connect RPC handlers. All follow **Fetch-Check-Act**:
   - **FETCH**: Read data via Reader services (non-blocking, parallel)
   - **CHECK**: Validate permissions/rules (pure Go, in memory)
   - **ACT**: Write via Writer services inside a short transaction
   - Publishes events to `eventstream` after successful transactions
2. **Service Layer** (`packages/server/pkg/service`) — Domain logic, split into Reader (read-only, `*sql.DB` pool) and Writer (write-only, `*sql.Tx`, per-transaction)
3. **Model Layer** (`packages/server/pkg/model`) — Pure Go domain structs bridging API (Proto) and DB (sqlc) types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-dev-tools/dev-tools](https://github.com/the-dev-tools/dev-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
