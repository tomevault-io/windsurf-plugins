---
trigger: always_on
description: 1. **This file** — overview and quick reference
---

# RouterOS LSP — Workspace Instructions

## Reading Order

1. **This file** — overview and quick reference
2. **[CLAUDE.md](../CLAUDE.md)** — full architecture, design decisions, and patterns
3. **[DESIGN.md](../DESIGN.md)** — design rationale and cross-project patterns
4. **[BACKLOG.md](../BACKLOG.md)** — active work index only
5. **`.github/instructions/`** — file-scoped rules loaded automatically by `applyTo` globs

## Documentation Ownership

- **BACKLOG.md** — active, actionable work only. Completed history and durable
  research findings should move out and leave only links to the source of truth.
- **docs/** — stable user/developer references and research write-ups.
- **.github/instructions/** — scoped agent/process rules tied to files and
  workflows.
- **DESIGN.md** — accepted design decisions and tradeoffs.
- **CHANGELOG.md** — shipped user-visible release notes, not live task tracking.

## Agent Roles

This project is used by thousands of VSCode users — the bar is higher than for other tikoci projects. Two agent systems share the work:

- **GitHub Copilot (primary)** — implementation, routine edits, PR authoring. Runs both in the IDE and as hosted agents on github.com, which means changes must not assume a local dev environment is present.
- **Claude Code (secondary)** — design work (DESIGN.md, architectural decisions), code review on PRs, deeper refactors. Claude is invoked deliberately; Copilot is the default.

Both agents read this file and CLAUDE.md — keep instructions model-agnostic. When something only applies to one, label it inline (e.g. `Copilot:` / `Claude:`).

## Project Overview

RouterOS LSP is a Language Server Protocol implementation for MikroTik RouterOS scripts. It queries a **live RouterOS device** via `/console/inspect` REST API to provide syntax highlighting, completion, diagnostics, hover, and symbols.

**Three build targets, six deployment contexts.** The build targets collapse to one codebase; the deployment contexts are where the user-facing failure modes live. Track both.

| Build target | Deployment context | Entry | Transport | First-class artifact |
|---|---|---|---|---|
| Client+Server CJS | VSCode Desktop | `extension.ts` → `server.ts` | Node IPC | `package.json` + VSIX |
| Web bundles | VSCode Web (`vscode.dev`, `github.dev`) | `extension.web.ts` → `server.web.ts` | Web Worker | webpack bundles |
| Server CJS | npm package `@tikoci/routeroslsp` | `server.ts` | stdio (via `routeroslsp` bin) | `server/package.json` |
| Native binary | Standalone download (Helix, other LSP clients) | `server.ts --stdio` | stdio / `--socket=<port>` | `build-standalone.sh`, GitHub Releases |
| Native binary / npm | **NeoVim** | `server.ts --stdio` | stdio | `nvim-routeros-lsp-init.lua` |
| Server CJS | GitHub Copilot CLI | `server.ts` | stdio, reads `initializationOptions` / `ROUTEROSLSP_*` env fallback | `.github/lsp.json` |

NeoVim shares the transport with "generic standalone" but has its own first-class config artifact, its own README section, and its own `neovim.instructions.md` — changes to the standalone path must be validated against NeoVim explicitly.

Full per-context gotchas and pre-release checks: [`deployment.instructions.md`](instructions/deployment.instructions.md).

## Build & Dev

```bash
bun install                 # Install all deps (chains client + server)
bun run compile             # Full build (client + server + exe + web)
bun run watch:node          # Dev mode — rebuild server on changes
bun run test                # Run all tests (unit + snapshot + model + client)
bun run lint                # Biome + typecheck
bun run vsix:package        # Package .vsix for VSCode Marketplace
bun run bun:exe             # Build standalone binary (copies to ~/.bin/)
```

**F5** in VSCode launches the Extension Development Host (preLaunchTask: `compile`).

### CI vs Release

Two workflows in `.github/workflows/`:

- **`ci.yaml`** — runs on every push to `main` and on pull requests. Does install → compile → test → lint → smoke. **No release artifacts, no publish.** This is the gate that catches regressions between releases. If it goes red, fix it before more changes pile on.
- **`build.yaml`** — manual `workflow_dispatch` only. Re-runs the same gate, then cross-compiles the 8 standalone binaries, packages VSIX, creates the GitHub Release, and publishes to Marketplace + Open-VSX + npm. Releasing stays an explicit human action.

When changing build scripts, lint config, the test runner, or the smoke harness, update both workflows so the CI gate matches the release gate.

## Critical Conventions

- **Bun over Node.js** — `bun install`, `bun run`, `bun build`, `bun test`
- **Tabs** for indentation, **single quotes** for strings (see `.vscode/settings.json`)
- **No `console.log`** in extension code — use LSP `connection.console` or `log.*` helpers in `shared.ts`
- **Server is the brain** — LSP features belong in `server/`, not `client/`. The client is a thin VSCode binding that proxies to the server. Don't replace LSP protocol with VSCode-specific APIs.
- **All syntax data comes from RouterOS** — the LSP has no built-in grammar. Every completion, diagnostic, and token comes from querying a live device's `/console/inspect` endpoint.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tikoci/lsp-routeros-ts](https://github.com/tikoci/lsp-routeros-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
