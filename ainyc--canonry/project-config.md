---
trigger: always_on
description: `canonry` is an **agent-first** open-source AEO operating platform that tracks how AI answer engines cite a domain for tracked keywords and acts on the signal through the content engine and integrations. Published as `@ainyc/canonry` on npm. The CLI and API are the primary interfaces — the web dashboard is supplementary.
---

# AGENTS.md

## Project Overview

`canonry` is an **agent-first** open-source AEO operating platform that tracks how AI answer engines cite a domain for tracked keywords and acts on the signal through the content engine and integrations. Published as `@ainyc/canonry` on npm. The CLI and API are the primary interfaces — the web dashboard is supplementary.

## Workspace Map

```text
apps/api/                        Cloud API entry point (imports packages/api-routes)
apps/worker/                     Cloud worker entry point
apps/web/                        Vite SPA source (bundled into packages/canonry/assets/)
packages/canonry/                Publishable npm package (CLI + server + bundled SPA)
packages/api-routes/             Shared Fastify route plugins
packages/contracts/              DTOs, enums, config-schema, error codes
packages/config/                 Typed environment parsing
packages/db/                     Drizzle ORM schema, migrations, client (SQLite/Postgres)
packages/provider-gemini/        Gemini adapter
packages/provider-openai/        OpenAI adapter
packages/provider-claude/        Claude/Anthropic adapter
packages/provider-local/         Local LLM adapter (OpenAI-compatible API)
packages/provider-perplexity/    Perplexity adapter
packages/provider-cdp/           Chrome DevTools Protocol adapter
packages/integration-google/     Google Search Console integration
packages/integration-google-analytics/  Google Analytics 4 integration
packages/integration-bing/       Bing Webmaster Tools integration
packages/integration-wordpress/  WordPress integration
docs/                            Architecture, roadmap, testing, ADRs
```

Start with `docs/README.md` when you need the current doc map, active plans, ADR index, or canonical roadmap.

## Commands

```bash
# One-command dev setup: install deps, build all packages, install canonry globally
./canonry-install.sh

pnpm install
pnpm run typecheck
pnpm run test
pnpm run lint
pnpm run dev:web

# CLI
canonry init
canonry serve
canonry project create <name> --domain <domain> --country US --language en
canonry keyword add <project> <keyword>...
canonry keyword replace <project> <keyword>...
canonry competitor add <project> <domain>...
canonry competitor remove <project> <domain>...
canonry run <project>
canonry run <project> --provider gemini          # single-provider run
canonry status <project>
canonry apply <file...>                          # multi-doc YAML + multiple files
canonry export <project>

# Agent layer
canonry agent ask <project> "<prompt>"               # one-shot turn against built-in Aero
canonry agent ask <project> "<prompt>" --provider zai --format json
canonry agent attach <project> --url <webhook-url>   # subscribe an external agent to run/insight events
canonry agent detach <project>                       # remove the agent webhook
canonry agent memory list <project>                  # list Aero's durable project-scoped notes
canonry agent memory set <project> --key <k> --value <v>    # upsert a note (2 KB max)
canonry agent memory forget <project> --key <k>      # delete a note

# Doctor — health checks (extensible registry: google.auth.*, ga.auth.*, config.providers, …)
canonry doctor                                                # global checks (provider keys, etc.)
canonry doctor --project <name>                               # project-scoped checks (Google/GA auth, redirect URI, scopes)
canonry doctor --project <name> --check google.auth.* --format json   # filter by id/wildcard, JSON output

# MCP adapter (separate bin, stdio only)
canonry-mcp                                          # core tier (~12 tools); load toolkits on demand
canonry-mcp --read-only                              # core read tier; toolkits load read-only tools only
canonry-mcp --eager                                  # register all 51 API tools at startup (legacy flat catalog)

# MCP client install helpers (operate on local client config files)
canonry mcp install --client claude-desktop          # merges a canonry entry into the config
canonry mcp install --client cursor --read-only      # scope to the 35 read tools
canonry mcp config  --client codex                   # print snippet for clients without auto-install

# Skills — install canonry's agent playbook into a user's project
canonry skills list                                  # show bundled skills (canonry-setup, aero)
canonry skills install                               # write both skills into ./.claude/skills/ + ./.codex/skills/ (default)
canonry skills install aero --client claude          # install only the analyst skill, no codex symlink
canonry skills install --dir ~/projects/foo --force  # custom target, overwrite divergent local edits
```

## Agent Layer

Canonry ships a built-in AI agent called **Aero**, backed by
[`@mariozechner/pi-agent-core`](https://github.com/badlogic/pi-mono). Aero
is an AEO analyst: it reads project state, analyzes regressions, acts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AINYC/canonry](https://github.com/AINYC/canonry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
