---
trigger: always_on
description: Shared repository instructions for Codex and OpenCode. Claude Code uses `CLAUDE.md` instead.
---

# AGENTS.md

Shared repository instructions for Codex and OpenCode. Claude Code uses `CLAUDE.md` instead.

The `## Rules` section below is **auto-generated** from `.claude/rules/*.md` — do not edit it manually. Run `pnpm build:agents-md` to regenerate it.

## Monorepo Structure

pnpm workspaces + Turborepo. TypeScript frontend, Rust backend, and several Rust SDKs.

| Package | Stack | Purpose |
|---|---|---|
| `packages/desktop/` | Electron + React | Desktop shell and frontend (`@cadencr/desktop`) |
| `packages/service/` | Rust (axum, utoipa) | Backend API server; runs as Electron sidecar in packaged builds |
| `packages/claude-agent-sdk-rs/` | Rust | SDK for Claude Code agents |
| `packages/codex-app-server-sdk-rs/` | Rust | SDK for Codex agents |
| `packages/opencode-sdk-rs/` | Rust | SDK for OpenCode agents |
| `packages/cli-discovery/` | Rust | Detects locally installed agent CLIs |
| `packages/landing/` | Next.js | Marketing site, docs, roadmap |

## Agent Providers

Cadencr is provider-neutral by design. Each supported agent (Claude Code, OpenCode, Codex) has its own Rust SDK in `packages/*-sdk-rs/` that handles transport/protocol details only. Provider-specific business logic lives in adapters inside `packages/service/`; shared frontend and backend code consumes provider-neutral types and catalog data — never branch on provider identity in generic code.

## Workflow

Requires `pnpm`, Node `>=22.18.0 <23.0.0`, and `cargo-watch` for `pnpm dev`.

```bash
pnpm dev                                  # frontend + service via Turborepo (alias: pnpm start)
pnpm build                                # build the desktop app
pnpm test                                 # vitest (frontend) + cargo test (Rust)
pnpm lint                                 # oxlint
pnpm format                               # oxfmt + cargo fmt
pnpm --filter @cadencr/desktop ts-check   # TypeScript type-check
pnpm --filter @cadencr/desktop knip       # unused-export detection
```

Target a single package: `pnpm --filter @cadencr/desktop <task>`. Frontend/service ports are configured via `packages/desktop/.env` and `packages/service/.env` (defaults `1420` / `5005`).

## Architecture

Electron desktop shell with a React frontend. The backend is the Rust API server in `packages/service/`, spawned as a sidecar in production; in dev `pnpm dev` runs it alongside the frontend via Turborepo. Frontend ↔ backend communication is HTTP (Axios) for requests and WebSocket (Zustand store) for streaming updates. Folder selection uses Electron native dialogs through the preload bridge.

Frontend path alias: `@` → `packages/desktop/src/` (for example `import { foo } from "@/lib/foo"`).

## Design System

`DESIGN.md` is the source of truth for Cadencr Desktop visual design: tokens, themes, typography, layout states, component anatomy, iconography, and UI self-audit checks.

- Before changing frontend UI, layout, styling, design tokens, icons, or user-facing visual behavior, read `DESIGN.md` and preserve its constraints.
- Do not load or summarize `DESIGN.md` for backend-only, SDK-only, migration-only, or non-visual documentation work.
- If implementation and `DESIGN.md` conflict, pause and surface the mismatch instead of silently inventing a new visual rule.

## Project-specific workflows

**Regenerating the API client.** After changing the Rust API surface (utoipa attributes / new handlers), run `pnpm --filter @cadencr/desktop run generate:api`. This re-emits `packages/service/openapi.json` (gitignored, derived from utoipa) and regenerates `packages/desktop/src/api/generated/index.ts` via orval — commit the regenerated TS file. Naming overrides for hooks live in `packages/desktop/orval.transformer.cjs`.

## Scoped Rules

Additional scoped rules for specific directories:

- `packages/desktop/src/AGENTS.md`
- `packages/desktop/src/components/AGENTS.md`
- `packages/desktop/src/routes/AGENTS.md`
- `packages/service/migrations/AGENTS.md`

## Shared Skills

Project-specific skills use agent-skills-compatible directories:

- Codex and OpenCode can load `.agents/skills/*/SKILL.md`
- Claude Code loads `.claude/skills/*/SKILL.md`

If a task clearly matches one of these skills, read the matching skill and follow it before editing:

- `db`
- `migration-safety`
- `qa`
- `finish-job`

## Command Aliases

- `/qa [feature]`: run the QA workflow from `.agents/skills/qa/SKILL.md`
- `/finish-job [scope or notes]`: simplify the current implementation, close test coverage gaps, propose a commit plan, wait for approval, then execute the safe commit flow

For agents that do not support project slash commands natively, treat these as semantic aliases and follow the mapped skill. For Codex specifically, if `/finish-job` appears in a prompt treat it as a plain-language alias for the `finish-job` skill in `.agents/skills/finish-job/`.

## Rules

<!-- begin:rules -->

### components

shadcn/ui components go in `ui/` subdirectory (new-york style, neutral base). Custom components go directly in `components/`.

### database


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [merkr-software/CadencR](https://github.com/merkr-software/CadencR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
