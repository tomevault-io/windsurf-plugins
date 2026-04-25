---
trigger: always_on
description: MeshClaw — OpenClaw channel plugin for Meshtastic LoRa mesh networks. Enables
---

# AGENTS.md — MeshClaw

## Project Overview

MeshClaw — OpenClaw channel plugin for Meshtastic LoRa mesh networks. Enables
sending/receiving messages over Meshtastic devices via USB serial, HTTP (WiFi), or MQTT broker.

- **Language**: TypeScript (strict mode, no `tsconfig.json` — OpenClaw host loads TS via esbuild)
- **Module system**: ESM (`"type": "module"` in package.json)
- **Runtime**: Node.js 22+, executed by the OpenClaw plugin host (not standalone)
- **Schema validation**: Zod v4 (not v3 — different API: `z.object().strict()`, no `.passthrough()`)
- **Key deps**: `@meshtastic/core`, `@meshtastic/transport-http`, `@meshtastic/transport-node-serial`, `mqtt`, `zod`
- **Repo**: `github.com/Seeed-Solution/MeshClaw` (npm: `@seeed-studio/meshtastic`)

## Build / Lint / Test Commands

There is **no build step, no linter, no test suite**. The plugin is loaded directly
from TypeScript source by the OpenClaw runtime via the `openclaw.plugin.json` manifest.

```bash
npm install                                    # Install dependencies
openclaw plugins install -l ./MeshClaw  # Install plugin locally for dev
openclaw channels status --probe               # Verify plugin is working

# Restart gateway after code changes (foreground process)
kill -9 $(pgrep -f "openclaw-gateway" | head -1); sleep 2; openclaw gateway &

# No build, lint, or test commands exist.
# If adding tests, use vitest (ESM-native) and place tests in src/__tests__/.
```

## CI / Workflows

- **Publish** (`publish.yml`): `npm publish --provenance` on `v*` tags. No build/test gate.
- **Translate README** (`readme-translate.yml`): Translates `README.md` into 5 languages
  (zh-CN, ja, fr, pt, es) via LLM API. Triggered by `workflow_dispatch` or `/translate`
  comment on PRs. Config lives in `.github/translate/` (languages, glossaries, prompts).
  Translation script: `.github/scripts/translate_readme.py` (Python, uses streaming SSE).

## Project Structure

```
index.ts                  # Plugin entry — default export, registers channel with OpenClaw
openclaw.plugin.json      # Plugin manifest (id, channels, configSchema)
src/
  types.ts                # Shared type definitions (config, messages, probes)
  config-schema.ts        # Zod v4 schemas for config validation
  runtime.ts              # Singleton plugin runtime accessor (module-level mutable ref)
  channel.ts              # Main ChannelPlugin implementation (the "glue" file)
  accounts.ts             # Multi-account config resolution and merging
  client.ts               # Serial/HTTP device connection via @meshtastic/core
  mqtt-client.ts          # MQTT broker connection via mqtt.js
  monitor.ts              # Gateway lifecycle — starts/stops device or MQTT monitors
  inbound.ts              # Inbound message processing (access control, routing, dispatch)
  send.ts                 # Outbound message sending (stripMarkdown, resolves transport)
  policy.ts               # Group/DM access policy evaluation (allowlists, mention gates)
  normalize.ts            # Node ID normalization, target parsing, allowlist matching
  onboarding.ts           # Interactive setup wizard integration
.github/
  workflows/              # CI: publish.yml, readme-translate.yml
  scripts/                # translate_readme.py — LLM-based README translation
  translate/              # languages.json, glossary/, prompts/, do-not-translate.md
```

## Code Style

### Formatting

- 2-space indentation, double quotes, semicolons always
- Trailing commas in multi-line arrays, objects, and parameter lists
- Soft line length ~110 characters
- No prettier or eslint config — maintain consistency manually

### Imports

- **Always** use `.js` extension for local imports: `import { foo } from "./bar.js"`
- Separate `import type` from value imports — never mix in one statement
- Type-only imports use the `type` keyword: `import type { Foo } from "./types.js"`
- Inline `type` keyword when mixing value + type in one import: `import { foo, type Bar } from "..."`

**Import order** (blank line between groups):
1. Node built-ins (`node:crypto`, `node:fs/promises`)
2. Third-party packages (`@meshtastic/core`, `mqtt`, `zod`)
3. OpenClaw SDK (`openclaw/plugin-sdk`)
4. Local modules (`./accounts.js`, `./types.js`) — alphabetical within group

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files | kebab-case | `config-schema.ts`, `mqtt-client.ts` |
| Functions | camelCase, domain-prefixed | `resolveMeshtasticAccount`, `normalizeMeshtasticNodeId` |
| Types | PascalCase, domain-prefixed | `MeshtasticAccountConfig`, `MeshtasticTransport` |
| Zod schemas | PascalCase, suffix `Schema` | `MeshtasticConfigSchema`, `MeshtasticAccountSchemaBase` |
| Constants | SCREAMING_SNAKE_CASE | `CHANNEL_ID`, `MESHTASTIC_CHUNK_LIMIT` |
| Enum-like objects | PascalCase name, `as const` | `DeviceStatus = { Connected: 5 } as const` |
| Booleans | `is`/`has`/`was` prefix | `isGroup`, `hasConfiguredGroups`, `wasMentioned` |

### Types & Type Safety

- Use `type` keyword (not `interface`) for ALL type definitions
- Shared types live in `src/types.ts`; module-local types are defined inline
- Use `satisfies` for type narrowing without widening

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Seeed-Solution/MeshClaw](https://github.com/Seeed-Solution/MeshClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
