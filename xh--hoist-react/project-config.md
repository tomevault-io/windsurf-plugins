---
trigger: always_on
description: This file provides guidance to AI coding assistants — including Claude Code, GitHub Copilot, and
---

# CLAUDE.md

This file provides guidance to AI coding assistants — including Claude Code, GitHub Copilot, and
similar tools — when working with code in this repository.

## Project Overview

Hoist-react is the client-side component of the Hoist web application development toolkit, built
by Extremely Heavy Industries (xh.io). It is a **library package** (not a standalone app) published
as `@xh/hoist` and consumed by Hoist application projects. The server-side counterpart is
[hoist-core](https://github.com/xh/hoist-core).

- **Language**: TypeScript
- **Framework**: React with MobX for reactive state management
- **Package manager**: Yarn

## Hoist Developer Tools and Documentation

**IMPORTANT: Do not guess at hoist-react APIs, component props, or framework patterns.** Hoist-react
ships dedicated tools that provide structured access to all framework documentation and TypeScript
type information. **You MUST use these tools before modifying or extending hoist-react code** to
understand existing architecture, configuration patterns, and common pitfalls. The package READMEs
and concept docs are the authoritative reference for how Hoist works -- skipping them risks producing
code that conflicts with established patterns or misses built-in functionality.

Two interfaces are available. Both share the same underlying registries and produce identical output:

**MCP Server (hoist-react)** -- When working in the hoist-react repository, an MCP server is
configured via `.mcp.json` and is very likely already available. Use the `hoist-search-docs`,
`hoist-list-docs`, `hoist-read-doc`, `hoist-search-symbols`, `hoist-get-symbol`, and
`hoist-get-members` tools, plus `hoist://docs/{id}` resources for direct document access.

**CLI Tools** -- For environments without MCP support, or when you prefer shell commands. These are
real `bin` entries in the hoist-react `package.json` — invoke them exactly as shown with `npx`:

```bash
# Documentation
npx hoist-docs search "grid sorting"         # Search all docs by keyword
npx hoist-docs read cmp/grid                 # Read a specific doc by ID
npx hoist-docs list                          # List all available docs
npx hoist-docs conventions                   # Print coding conventions
npx hoist-docs index                         # Print the documentation catalog

# TypeScript symbols and types
npx hoist-ts search GridModel                # Search for symbols and class members
npx hoist-ts symbol GridModel                # Get detailed type info for a symbol
npx hoist-ts members GridModel               # List all members of a class/interface
```

**Use `search` for discovery** — it matches against symbol names, JSDoc content, and own member
names. Multi-word queries use AND logic (e.g. `"panel modal"` finds ModalSupportModel via its
JSDoc, `"StoreRecord raw"` finds StoreRecord via its `raw` property). Also searches public members
of every exported class and every exported `*Config` interface (e.g. `GridConfig`, `StoreConfig`)
by owner name, member name, and JSDoc — so a query for `"groupSortFn"` reaches both `GridModel`
and `GridConfig`. Use `symbol` and `members` when you already know the exact PascalCase name.
When multiple symbols share a name (e.g. `View` exists in both `cmp/viewmanager` and `data/cube`),
pass the file path to `symbol` or `members` to disambiguate — the tools will hint when this is
needed. Run `npx hoist-docs --help` and `npx hoist-ts --help` for full usage.

**Recommended workflow:** Start with the documentation index (`hoist-docs index` or `hoist://docs/index`)
to discover available docs. Use the "Quick Reference by Task" table to find the right doc for your
goal, then read the relevant README(s). Supplement with TypeScript symbol lookups for precise API
details. The docs provide architectural context and common pitfalls; the TypeScript tools provide
exact signatures, decorators, and member listings.

### GitHub MCP Server (opt-in)

A Docker-based server providing GitHub API tools (issues, PRs, code search, etc.) via the
official `github-mcp-server` image. Configured in `.mcp.json` but **not enabled by default** —
it requires Docker and an authenticated GitHub CLI, which not every developer keeps running.

**To enable:**

1. Install and start **Docker**.
2. Install the **GitHub CLI** (`brew install gh`) and authenticate with `gh auth login`. The
   server invokes `gh auth token` at startup to fetch a token from the macOS Keychain (or
   `gh`'s credential store on other platforms), so no plaintext token needs to live in your
   shell environment.
3. Add `"github"` to `enabledMcpjsonServers` in `.claude/settings.local.json` (local settings
   merge with the shared `settings.json` — enabling locally does not affect other developers):
   ```json
   {
     "enabledMcpjsonServers": ["hoist-react", "github"]
   }
   ```

If Docker is not running or `gh` is not authenticated when the server is enabled, Claude Code
may show errors on startup — remove `"github"` from your local settings to resolve.

**Fallback when not enabled:** The `gh` CLI provides functionally equivalent access to the same
operations (`gh pr view`, `gh issue list`, `gh api`, `gh pr create`, etc.). Prefer `gh` over
crafting raw `curl` calls to the GitHub API.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xh/hoist-react](https://github.com/xh/hoist-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
