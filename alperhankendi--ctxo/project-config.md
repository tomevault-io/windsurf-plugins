---
trigger: always_on
description: Ctxo is a **Model Context Protocol (MCP) server** that gives AI coding assistants dependency-aware, history-enriched context for codebases. It delivers "Logic-Slices" — a symbol plus all transitive dependencies, git intent, anti-pattern warnings, and change health scores — in under 500ms.
---

# CLAUDE.md — Ctxo

## Project Overview

Ctxo is a **Model Context Protocol (MCP) server** that gives AI coding assistants dependency-aware, history-enriched context for codebases. It delivers "Logic-Slices" — a symbol plus all transitive dependencies, git intent, anti-pattern warnings, and change health scores — in under 500ms.

* **Type:** npm package / CLI tool / MCP server (stdio transport)
* **Author:** Alper Hankendi
* **Status:** v0.7.0-alpha.0 — 14 MCP tools, pnpm monorepo with 5 packages, 987+ tests
* **Language:** TypeScript (ESM-first, `"type": "module"`)
* **Runtime:** Node.js >= 20

## Quick Reference

```Shell
# Dev (pnpm workspace)
pnpm --filter @ctxo/cli dev       # zero-build dev run
pnpm -r build                     # build all packages
pnpm -r test                      # run tests across workspace (987+ tests)
pnpm -r typecheck                 # typecheck all packages
pnpm --filter @ctxo/cli test:unit # unit tests only
pnpm --filter @ctxo/cli test:e2e  # end-to-end tests
pnpm --filter @ctxo/cli build     # build CLI package to dist/

# Usage (consumer)
npx ctxo install                     # install language plugins (interactive)
npx ctxo install typescript go --yes # non-interactive install of specific plugins
npx ctxo install --dry-run --pm pnpm # preview install plan with chosen pm
npx ctxo index                       # build codebase index
npx ctxo index --install-missing     # auto-install detected plugins, then index
npx ctxo index --check               # CI gate: fail if index stale
npx ctxo index --skip-history        # fast re-index without git history
npx ctxo index --max-history 5       # limit commit history per file
npx ctxo watch                       # file watcher for incremental re-index
npx ctxo sync                        # rebuild SQLite from committed JSON
npx ctxo init                        # install git hooks + language detection/install prompt
npx ctxo init --no-install           # init without plugin install prompt
npx ctxo status                      # show index manifest
npx ctxo doctor                      # health check all subsystems (--json, --quiet)
npx ctxo doctor --fix                # dependency-ordered remediation (--dry-run, --yes)
npx ctxo version                     # verbose version report (default)
npx ctxo --version --verbose         # version + installed plugins list
npx ctxo --version --json            # machine-readable version report
npx ctxo visualize                   # generate interactive dependency graph HTML
npx ctxo visualize --max-nodes 200   # limit to top 200 symbols by PageRank
npx ctxo visualize --no-browser      # skip auto-opening browser

# Environment
DEBUG=ctxo:*                  # enable all debug output
DEBUG=ctxo:git,ctxo:storage   # enable specific namespaces
CTXO_RESPONSE_LIMIT=16384     # response truncation threshold (default 8192)
```

## Architecture

**pnpm monorepo + Hexagonal (Ports & Adapters)** — strict dependency direction inside `@ctxo/cli`:

```
packages/
├── cli/              @ctxo/cli             # primary CLI + MCP server (composition root)
├── plugin-api/       @ctxo/plugin-api      # plugin protocol v1 types (no runtime deps on cli)
├── lang-typescript/  @ctxo/lang-typescript # ts-morph, full tier
├── lang-go/          @ctxo/lang-go         # tree-sitter Go
└── lang-csharp/      @ctxo/lang-csharp     # Roslyn + tree-sitter + tools/ctxo-roslyn
```

### Key Rules

* **Hexagonal boundaries hold inside** **`@ctxo/cli`:**
  * **`core/`** **NEVER imports from** **`adapters/`** — pure domain logic only
  * **`core/`** **NEVER imports from** **`ports/`** — ports import from core types
  * **`ports/`** **NEVER imports from** **`adapters/`**
  * **`adapters/mcp/`** **may import from** **`core/`** **and** **`ports/`** **only**
  * **Composition root (`packages/cli/src/index.ts`)** is the ONLY file that wires adapters to ports
* **Plugins (`@ctxo/lang-*`) import only from** **`@ctxo/plugin-api`; never from** **`@ctxo/cli`.**
* **No barrel exports** (`index.ts` re-exports) — import directly from source file path
* **Port-first rule:** every adapter MUST implement a port interface

### Project Structure (inside `packages/cli/`)

```
packages/cli/src/
  index.ts                     # composition root + StdioServerTransport
  ports/                       # interfaces only (ILanguageAdapter, IStoragePort, IGitPort, IMaskingPort)
  core/                        # pure domain (graph, logic-slice, blast-radius, overlay, why-context, change-intelligence, masking, detail-levels)
    detection/detect-languages.ts
    install/package-manager.ts
  adapters/
    language/                  # plugin loader facade (ts-morph/Go/C# live in sibling packages)
      plugin-discovery.ts      # scans project package.json for @ctxo/lang-*, ctxo-lang-*
    storage/                   # SQLite cache + JSON index read/write
    git/                       # simple-git wrapper
    watcher/                   # chokidar file watcher
    workspace/single-package-workspace.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alperhankendi/Ctxo](https://github.com/alperhankendi/Ctxo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
