---
trigger: always_on
description: This is a standalone OpenClaw plugin (`@apify/apify-openclaw-plugin`) that provides web scraping and data extraction via Apify's API. It registers **1 agent tool** (`apify`) — a universal scraper with 3 actions: `discover`, `start`, and `collect`.
---

# Apify OpenClaw Integration — Repository Guidelines

## Overview

This is a standalone OpenClaw plugin (`@apify/apify-openclaw-plugin`) that provides web scraping and data extraction via Apify's API. It registers **1 agent tool** (`apify`) — a universal scraper with 3 actions: `discover`, `start`, and `collect`.

- **Upstream repo:** https://github.com/openclaw/openclaw
- **Plugin docs:** https://docs.openclaw.ai/plugins/community
- **Agent tools guide:** https://docs.openclaw.ai/plugins/agent-tools
- **Plugin system docs:** https://docs.openclaw.ai/plugins
- **Apify console:** https://console.apify.com/
- **Support:** integrations@apify.com

## Project Structure

```
src/
  index.ts                    # Plugin entry point — registers apify + CLI
  apify-client.ts             # Shared Apify client factory, config helpers
  cli.ts                      # openclaw apify setup|status|test commands
  util.ts                     # Inlined utilities: ToolInputError, normalizeSecretInput, wrapExternalContent
  tools/
    apify-scraper-tool.ts     # Universal scraper — discover + start + collect
test/
  helpers.ts                  # makeMockFetch, standardRunResponses, TEST_CONFIG
  apify-scraper.test.ts       # Tool tests
openclaw.plugin.json          # Plugin manifest (configSchema + uiHints) — REQUIRED
package.json                  # npm package config
```

## The `apify` Tool

Single tool with 3 actions:

| Action | Purpose |
|--------|---------|
| `discover` + `query` | Search Apify Store for actors by keyword |
| `discover` + `actorId` | Fetch actor's input schema + README |
| `start` + `actorId` + `input` | Fire an actor run, returns runId/datasetId |
| `collect` + `runs` | Poll status, return results for completed runs |

### Tool Description Guidance

The tool description includes instructions for the agent:
- **Sub-agent delegation:** Tool should be used by a sub-agent that returns only relevant extracted data, not raw dumps.
- **Batching:** Batch multiple URLs into a single run (e.g. `startUrls: [{url: "..."}, ...]`).
- **Known actors:** Compact comma-separated list of 57 actors across Instagram, Facebook, TikTok, YouTube, Google Maps, and more.
- **Support:** Directs users to integrations@apify.com for issues.

## Key Architecture Decisions

- **Single tool, multiple actions:** All scraping goes through `apify` with `discover`/`start`/`collect` actions.
- **Async two-phase pattern:** `start` returns immediately with run references. `collect` polls and fetches results. The agent does other work between calls.
- **`apify-client` SDK:** Uses the official `apify-client` npm package (not raw HTTP). Client created via `createApifyClient(apiKey, baseUrl)`.
- **Inlined utilities (`util.ts`):** `ToolInputError`, `normalizeSecretInput`, and `wrapExternalContent` are NOT exported from `openclaw/plugin-sdk`. We carry local copies.
- **No build step:** OpenClaw loads plugins via `jiti` (TypeScript JIT). We ship `.ts` source directly.
- **No skills:** Skills were removed — the tool description and `discover` action provide all needed guidance.

## Apify Actor IDs

**Format: `username~actor-name`** (tilde separator, not slash).

The `~` format avoids URL path ambiguity. The `discover` action builds slugs as `${username}~${name}`.

## Setup Wizard — Direct Config Write

`openclaw apify setup` (in `src/cli.ts`) writes config directly to the OpenClaw config file.

It uses:
- `api.runtime.config.loadConfig()` → returns current `OpenClawConfig`
- `api.runtime.config.writeConfigFile(cfg)` → writes it back to disk

The wizard merges safely: preserves existing config, adds to `tools.alsoAllow` without duplicates, uses `group:plugins` when all tools are selected.

## Build, Test, and Development

- **Runtime:** Node 22+ (required by openclaw peer dependency).
- **Install:** `npm install`
- **Type-check:** `npx tsc --noEmit`
- **Test:** `npx vitest run`
- **Pack (dry run):** `npm pack --dry-run`
- **Current state:** 1 test file, 10 tests passing.

## Coding Style

- TypeScript (ESM). Prefer strict typing; avoid `any`.
- Tool names: `snake_case` (e.g., `apify`).
- Plugin id / config keys: `kebab-case` (e.g., `apify`).
- Keep files concise. Add comments for non-obvious logic.
- Tool schema guardrails: avoid `Type.Union`. Use `stringEnum` for string enums, `Type.Optional(...)` instead of nullable types.

---

## How the OpenClaw Plugin System Works

This section documents how OpenClaw discovers, loads, and runs plugins. Reference this when modifying the plugin or debugging integration issues.

### Plugin Lifecycle

```
Discovery → Manifest Loading → Config Validation → Module Loading → Registration → Tool Resolution
```

#### 1. Discovery

OpenClaw scans for plugins in strict precedence order:

1. **Config paths** (`plugins.load.paths`) — highest priority
2. **Workspace extensions** (`<workspace>/.openclaw/extensions/`)
3. **Global extensions** (`~/.config/openclaw/extensions/` or `~/.openclaw/extensions/`)
4. **Bundled extensions** (shipped with OpenClaw) — lowest priority


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apify/apify-openclaw-plugin](https://github.com/apify/apify-openclaw-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
