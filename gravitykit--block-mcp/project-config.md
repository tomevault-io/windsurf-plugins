---
trigger: always_on
description: validates + enriches                             parses/serializes blocks,
---

# AGENTS.md — GK Block API + Block MCP

> Block-level WordPress content CRUD for AI agents, delivered as a two-part system: an npm-published MCP server (`@gravitykit/block-mcp`) that talks to a companion WordPress plugin (`gk-block-mcp`) over a private REST namespace. The plugin also ships a one-click **Connect** onboarding flow that provisions a dedicated, least-privilege agent account and hands the MCP client a credential out-of-band — no copy-pasting Application Passwords.

## Quick Start

**What this is:** Two components that ship and version independently.
1. **MCP server** (`src/`, built to `dist/index.cjs`) — a thin stdio MCP server that exposes ~30 tools, validates input, calls the plugin's REST API, and enriches responses with AI-friendly guidance (tier groupings, legacy warnings, error translation). Also contains the **connector** (`src/connect.ts`) — the `block-mcp connect` subcommand that drives a browser-Approve handshake and writes the client's MCP config.
2. **WordPress plugin** (`wordpress-plugin/gk-block-mcp/`) — registers the `gk-block-api/v1` REST namespace, owns block parsing/serialization/mutation, the preference/scoring engine, post/term/media lifecycle, and the Connect onboarding UI + credential flow.

**Main entry points:**
- MCP server: `src/index.ts` → built to `dist/index.cjs`; npm `bin` is `block-mcp`.
- Connector: `src/connect.ts` (`block-mcp connect …`).
- Plugin: `wordpress-plugin/gk-block-mcp/gk-block-mcp.php` (bootstraps on `rest_api_init` + admin hooks).

**Architecture style:** Hook-driven WordPress plugin (autoload via `spl_autoload_register`) behind a typed TypeScript MCP server. No framework; plain WP + the plain MCP SDK.

**Key namespaces:** PHP `GravityKit\BlockMCP`; REST `gk-block-api/v1`; npm `@gravitykit/block-mcp`.

**Versions:** the MCP server and plugin version independently. `package.json` `version` is the server; `readme.txt` `Stable tag` + `gk-block-mcp.php` `Version`/`GK_BLOCK_MCP_VERSION` are the plugin (current: 2.0.0). See **Versioning & Releases**.

```bash
cd MCPs/block-mcp && npm install
npm run build          # esbuild → dist/index.cjs
export WORDPRESS_URL=… WORDPRESS_USER=… WORDPRESS_APP_PASSWORD=…
npm start              # node dist/index.cjs (stdio)
```
The plugin must be active on the target site for the server to reach `gk-block-api/v1`.

## Repository Map

```text
MCPs/block-mcp/
├── AGENTS.md / CLAUDE.md (→ @AGENTS.md) / README.md
├── package.json                 # @gravitykit/block-mcp; esbuild build; bin: block-mcp
├── tsconfig.json                # ES2022, bundler resolution
├── src/
│   ├── index.ts                 # MCP server entry — aggregates tools, routes calls
│   ├── connect.ts               # `block-mcp connect` — loopback browser-Approve handshake + config write
│   ├── client.ts                # WordPressBlockClient — typed HTTP wrapper (axios, Basic Auth)
│   ├── types.ts                 # All TS interfaces
│   ├── enrichers.ts             # Syntax-highlight / response enrichment (shiki/core)
│   ├── error-translator.ts      # Maps REST error codes → actionable agent hints
│   ├── instructions.ts          # MCP server instructions / handshake addendum
│   ├── preferences.ts           # Client-side preference annotation (mirrors PHP)
│   └── tools/                    # discovery, read, write, mutate, patterns, posts, terms, media, yoast
├── dist/index.cjs               # Built bundle (esbuild, single CJS file) — shipped to npm
├── tests/                       # Vitest tests for the server + connector (connect.*.test.ts)
└── wordpress-plugin/gk-block-mcp/
    ├── gk-block-mcp.php          # Bootstrap: autoloader, rest_api_init wiring, admin wiring, CLI
    ├── uninstall.php             # Full data + agent teardown (multisite-aware)
    ├── readme.txt                # Canonical changelog + Upgrade Notice (WordPress plugin readme)
    ├── phpcs.xml.dist            # WordPress-Extra + WordPress-Docs + PHPCompatibilityWP (testVersion 7.4-)
    ├── phpstan.neon.dist         # PHPStan level 5, analyze-as-PHP-8.2, WP stubs
    ├── phpstan-bootstrap.php     # Placeholder constants for static analysis
    ├── includes/                 # 22 classes (see Core Classes)
    └── tests/                    # PHPUnit suite (SQLite drop-in) — see tests/AGENTS.md
```

## Architecture

### Two-Component Design

```text
AI client  ──stdio──▶  MCP server (TypeScript)  ──HTTPS Basic Auth──▶  WordPress plugin (PHP)
                       src/index.ts                                     wordpress-plugin/gk-block-mcp/
                       validates + enriches                             parses/serializes blocks,
                                                                        manages revisions, enforces
                                                                        preferences, owns the data
```

The MCP server is a thin orchestration layer: it validates inputs, delegates to the REST API via `WordPressBlockClient` (`src/client.ts`), and annotates responses (`src/enrichers.ts`, `src/preferences.ts`, `src/error-translator.ts`). All heavy lifting — block parsing, serialization, mutation, safety checks, rate limiting, revision tracking — lives in the PHP plugin.

### Plugin initialization flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GravityKit/block-mcp](https://github.com/GravityKit/block-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
