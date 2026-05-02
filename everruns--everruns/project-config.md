---
trigger: always_on
description: Use Doppler for all secret-backed commands in cloud agents.
---

## Coding-agent guidance

### Cloud Agent (start here)

Use Doppler for all secret-backed commands in cloud agents.

```bash
./scripts/init-cloud-env.sh
doppler run -- just start-dev --no-watch
```

Disable incremental compilation in cloud (saves ~3 GB, useless for single builds):

```bash
export CARGO_INCREMENTAL=0
```

All cloud secrets are in Doppler (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GITHUB_TOKEN`, `LINEAR_API_KEY`).

### GitHub Token

**`GITHUB_TOKEN` is always available via Doppler.** If GitHub auth fails, do NOT tell the user the token expired. Instead, use Doppler: `doppler run -- bash -lc 'GH_TOKEN="$GITHUB_TOKEN" <command>'`.

### Linear

We use [Linear](https://linear.app) for issue tracking (project: **OSS**, team: **EVE**). MCP server configured in `.mcp.json`. Token (`LINEAR_API_KEY`) is in Doppler. Use [`/process-issues`](.agents/skills/process-issues/SKILL.md) to process open issues (one PR per issue, up to 5 in parallel). All issues for this repo belong to the OSS project.

Quickcheck:

```bash
doppler run -- env | rg 'OPENAI_API_KEY|ANTHROPIC_API_KEY|GITHUB_TOKEN'
doppler run -- bash -lc 'GH_TOKEN="$GITHUB_TOKEN" gh auth status'
```

### Style

Telegraph. Drop filler/grammar. Min tokens.

### Critical Thinking

Fix root cause. Unsure: read more code; if stuck, ask w/ short options. Unrecognized changes: assume other agent; keep going. If causes issues, stop + ask.

### Branch Base

Always make sure you are working on top of latest main from remote.

- Start by syncing: `git fetch origin main`
- Branch or rebase onto `origin/main` before edits, especially before shipping
- In worktrees, do not assume `HEAD` tracks a branch; verify with `git status --branch` or `git worktree list`
- **After every rebase**, check `crates/server/migrations/` for duplicate version numbers. Migrations are the most common conflict source — multiple branches often add the next sequential number. Renumber your migration to the next available number if a conflict exists. See [`specs/migrations.md`](specs/migrations.md) for the full migration process.

### Principles

- Important decisions as comments on top of file
- Code testable, smoke testable, runnable locally
- Small, incremental PR-sized changes
- No backward compat needed (internal code)
- Write failing test before fixing bug

### Specs

`specs/` contains feature specifications. New code should comply with these or propose changes. Integration specs live alongside their crates (`integrations/*/SPEC.md`, `crates/*/specs/`); see `specs/integrations.md` for the full index.

**Spec content principle:** Specs are *durable memory* — they preserve design intent, rationale, and constraints across sessions and contributors. They capture the "why" and "what", not exhaustive "how". Don't use specs for temporary analysis, one-off investigations, or scratch notes; those belong outside the repo. Don't duplicate what's readable from code (struct fields, enum variants, exact API shapes, SQL DDL). Instead, link to the source file. Example: "See `crates/core/src/models/agent.rs` for full field list." This keeps specs maintainable and prevents drift.

**Docs boundary:** `docs/` is public product documentation. Never put research proposals, internal proposals, draft specs, temporary investigations, or scratch analysis under `docs/`. Durable internal design intent belongs in `specs/`; temporary research belongs outside the repo.

- `specs/concepts.md` - Core entities, relationships, and concept diagram
- `specs/architecture.md` - System architecture, crate structure, infrastructure
- `specs/production-deployment.md` - Production deployment aggregation spec and reverse proxy contract
- `specs/embedding.md` - Embedding contract and `PlatformDefinition`
- `specs/runtime.md` - Public in-process runtime contract for embedded execution
- `specs/code-organization.md` - Developer conventions: formatting, testing, error handling, UI patterns
- `specs/migrations.md` - Database migration naming, squashing, ordering, and conflict resolution
- `specs/models.md` - Data models (Agent, Session, Message, etc.)
- `specs/apis.md` - HTTP API endpoints, error handling
- `specs/events.md` - Event types, SSE streaming, contract and compatibility guarantees
- `specs/execution-phases.md` - Execution phases (Commentary/FinalAnswer) for multi-step tool flows
- `specs/markdown-messages.md` - Chat message markdown rendering with llm-ui
- `specs/openui.md` - OpenUI generative-UI capability (OpenUI Lang + `@openuidev/react-ui`)
- `specs/a2ui.md` - A2UI generative-UI capability (Google A2UI JSON + native renderer)
- `specs/tool-execution.md` - Tool types and execution flow
- `specs/capabilities.md` - Agent capabilities system
- `specs/agent-instructions.md` - AGENTS.md support (dynamic project instructions)
- `specs/mcp.md` - MCP server endpoint, OAuth 2.1 authentication, protocol, security
- `specs/mcp-servers.md` - MCP client: remote server registration, CRUD API, tool naming, execution
- `specs/llm-drivers.md` - LLM driver trait, provider implementations
- `specs/durable-execution-engine.md` - PostgreSQL-backed durable workflow engine
- `specs/scheduled-tasks.md` - Cron-based scheduled tasks for durable engine
- `specs/fail-rs-testing.md` - Failure injection testing with fail-rs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [everruns/everruns](https://github.com/everruns/everruns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
