---
trigger: always_on
description: This file is the canonical guide for humans and coding agents working in this
---

# AGENTS.md - local-openfinance

This file is the canonical guide for humans and coding agents working in this
repository. Keep it accurate, but keep product and technical specifications in
`specs/` instead of growing this file.

## User setup (Grok, OpenClaw, and other agents)

If the human asked you to install, configure, first-run, or "make this work"
on their machine, stop reading this contributor guide and follow
`.agents/skills/setup/SKILL.md`. That skill installs the localhost web UI and
the Linux systemd or macOS launchd jobs that keep it serving and syncing.

This file is for people changing the code.

## Self-update protocol (required)

When you learn something durable about this project, update the relevant
documentation in the same change before finishing the task.

- If the new detail is agent guidance, workflow, conventions, tooling,
  operational steps, or a rule for how contributors should work, update this
  file.
- If the new detail is product behavior, UI behavior, API behavior, data shape,
  SQL schema behavior, command behavior, validation rules, or application
  architecture, update the best matching file under `specs/`.
- If no matching spec exists, create one and link it from `specs/README.md`.
- Do not duplicate existing guidance. Revise or remove stale bullets when
  behavior changes.

Before finishing any task that touches config, including schema, loader,
`config.example.json`, or config behavior, check that `README.md` documents
every field and constraint in `schemas/config.schema.json`. Update `README.md`
in the same change if it is missing or stale. Treat the schema as canonical.

## Project overview

`local-openfinance` uses the Banco MCP REST API to synchronize Brazilian
OpenFinance data locally into SQLite. It annotates and labels entries for cost
segmentation and reporting.

Annotation can use embeddings and classifiers, including local models through
Ollama. Reporting and other LLM features use Vercel AI SDK model APIs.

Implementation roadmap: `PLAN.md`.

## Specification map

Start with `specs/README.md` when changing application behavior.

Key specs:

- `specs/cli-commands.md` - user-facing CLI commands and shared flags.
- `specs/config-and-validation.md` - JSON artifacts, config validation, and
  prompt validation.
- `specs/database-and-state.md` - SQLite state, migrations, and database
  conventions.
- `specs/sync-openfinance.md` - Banco MCP sync behavior and normalized fields.
- `specs/llm-analysis.md` - report model provider behavior.
- `specs/web-app.md` - shared web shell, security, i18n, formatting, and route
  conventions.
- `specs/web-transactions-tab.md` - Transactions tab.
- `specs/web-credit-cards-tab.md` - Credit Cards tab.
- `specs/web-investments-tab.md` - Investments tab.
- `specs/web-loans-tab.md` - Loans tab.
- `specs/web-accounts-tab.md` - Accounts tab.
- `specs/web-connections-tab.md` - Connections tab.
- `specs/web-categories-tab.md` - Categories tab.
- `specs/web-labels-tab.md` - Labels tab.
- `specs/web-classify-triage-tab.md` - Classify triage tab.
- `specs/web-sync-tab.md` - Sync tab.
- `specs/intelligence.md` - named reports, markdown memory, and Reports tab.
- `specs/npm-publish.md` - package publishing behavior.

Existing focused specs:

- `specs/account-balance-over-time.md`
- `specs/credit-card-statement-correlation.md`
- `specs/investment-position-history.md`

## Repository layout

```text
local-openfinance/
├── banco-mcp-openapi.json     # Banco MCP REST contract
├── schemas/                   # Ajv JSON Schemas
├── specs/                     # Product and technical specifications
├── examples/                  # Topic configs and prompt directories
├── src/
│   ├── local-openfinance.ts   # CLI entrypoint
│   ├── env.ts                 # dotenv, import first
│   ├── commands/              # One CommandModule per subcommand
│   ├── config/                # Config loading and validation
│   ├── openfinance/           # HTTP client and sync engine
│   ├── db/                    # SQLite migrations and query layer
│   ├── annotation/            # Categories, embeddings, classify wizard
│   ├── transfers/             # Cross-account movement linking
│   ├── state/                 # Shared state helpers
│   ├── scoring/               # Embedding/classifier providers
│   ├── llm/                   # Model generation helpers and bundled prompts
│   ├── intelligence/          # Named reports, briefing, agent, charts, mail
│   ├── web/                   # Vite React client and Hono server
│   └── utils/                 # Paths, JSON, local time, helpers
├── tests/                     # Vitest unit tests
├── scripts/                   # Build, prepare, smoke, import scripts
├── PLAN.md                    # Engineering plan and SQL DDL
└── README.md                  # User documentation
```

## Tooling

- Use `nvm use` before running project commands. `.nvmrc` pins the required
  Node version, currently 26+ for built-in `node:sqlite`.
- SQLite uses `import { DatabaseSync } from 'node:sqlite'`; do not add
  `better-sqlite3` or another DB npm dependency.
- TypeScript extends `@tsconfig/strictest` with `"types": ["node"]`.
- pnpm v11 is the package manager. `packageManager` pins the Corepack version.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barbieri/local-openfinance](https://github.com/barbieri/local-openfinance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
