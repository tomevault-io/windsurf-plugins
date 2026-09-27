---
trigger: always_on
description: This file governs AI coding agent behavior in the `octopi` repository. **Read this file before making any code changes.**
---

# AGENTS.md

This file governs AI coding agent behavior in the `octopi` repository. **Read this file before making any code changes.**

---

## Project Overview

- **Project name**: `octopi`
- **One-line summary**: An embeddable agent engine for building AI-powered applications.
- **Core stack**: TypeScript / Node.js / Vitest
- **Package manager**: `npm`
- **Runtime Directory**: `~/.octopi/`
- **SQLite**: built-in `node:sqlite` (`DatabaseSync`); requires **Node.js >= 24**. Do not reintroduce `better-sqlite3`.

---

## Repository Layout

```
src/           Source code entry point
tests/         Test directory
docs/          Documentation
arch/          Architecture design documents (internal)
config/        Configuration
web/           Web runtime interface
data/          Data/Session storage
```

---

## Configuration Files

| File | Tracked? | Purpose |
|------|----------|---------|
| `octopi.schema.json` | yes | JSON Schema for editor autocomplete / validation |
| `octopi.example.json` | yes | Canonical template — keep in sync with Zod schema |
| `octopi.json` | **no** (gitignored) | Local runtime instance only |

**Do not commit or recreate a repo-root `octopi.json` for day-to-day work.** It shadows the workspace config when `loadConfig` resolves `./octopi.json` first.

Canonical runtime config lives at **`~/.octopi/octopi.json`** (`OCTOPI_HOME`).

```sh
# preferred
octopi serve start -c ~/.octopi/octopi.json
# or
cd ~/.octopi && octopi serve start
```

CLI helpers (`ensureInitialized` / `ensureDaemonConfig`) prefer `OCTOPI_HOME` over cwd. When changing config shape, update **both** `src/config-schema.ts` and `octopi.schema.json` / `octopi.example.json`.

### Runtime home layout (`OCTOPI_HOME`, default `~/.octopi`)

Scaffolded by `src/init.ts` (`initOctopi` / `ensureAgentDirs`). Keep init, types, schema, and docs aligned with this tree:

```
~/.octopi/
  octopi.json
  audit/
  plugins/
  sessions/             # JsonlSessionStore (sessionId 一等；唯一 runtime Session 后端)
    sessions.json       # meta 索引（含 lifecycle/endedAt）
    <id>.jsonl / <id>.state.json
  sessions.index.db     # 可重建检索投影（可选；非权威，见 arch/session-history-search.md）
  archives/             # 归档冷备 *.sessions.jsonl.gz
  agents/<id>/          # agent home
    AGENTS.md           # main persona (loaded first by loadPersona)
    persona/            # supplemental persona (*.md, numeric prefix for order)
    skills/             # skillDirectory target
  workspace/<id>/       # tool sandbox cwd
```

**Do not create `agents/<id>/memory/` or `agents/<id>/wisdom/` directories.** Memory / Cognition / Wisdom / Knowledge persist in a per-agent SQLite file via `AgentDatabase` (`src/harness/memory/sqlite/agent-db.ts`), not as sibling folders under home.

**Do not use `memory.extractor` ETL or `MemoryExtractionWiring`.** Memory write path is agent `memory_store` + `memory.steward.*` subsystems. See `docs/memory.md` and `arch/memory-system-redesign.md`.

**Do not reintroduce `SqliteSessionStore`.** Runtime sessions are Jsonl-only (`OCTOPI_HOME/sessions/`). `sessions.index.db` is a rebuildable search projection (FTS5+LIKE), never a second authority. History tools: `session_search` / `session_read` (Information 原文) vs `memory_search` (命题). Spec: `arch/session-history-search.md`.

---

## Architecture & Invariants

**Dependency Direction**: Outer -> Inner. `Core` has zero outer dependencies. **Never introduce a dependency from `Core` to `Harness`.**

### Architecture constitution (required reading)

- **Constitution**: [`docs/north-star.md`](docs/north-star.md) — long-term invariants **I1–I6** / **E1–E7**. Implementation and review **must not violate** these.
- **External docs**: `docs/` (constitution, architecture, contracts). **Internal design**: `arch/` (gitignored; implementation handoffs live here).
- **Development constraints** (non-exhaustive; full list in the constitution):
  - **I1**: Mutable run context lives only in **RunScope**; `Agent` is a template + substrate, not a session workspace.
  - **E1/E5**: Same `sessionId` runs are serialized; Loop stays stateless; production path uses per-run context.
  - **E2/E7**: Lock/lease key is `sessionId`; v1 uses in-process `InProcessSessionLock` — **do not assume it is valid across processes**.
  - **E3**: Memory/Wisdom/Cognition write **only** that agent’s stores.
  - **E4**: Compact key is `(sessionId, agentId)`; do not borrow another agent’s compact as default.
  - **E6/I3**: Session ACL effective rights = L0 ∩ role.max ∩ agent.max ∩ binding; `preferredAgentId` ≠ `primaryAgentId`; handoff is host-plane by default.
  - **I5**: Tool cwd policy is `toolIsolation` (default `none`); `session-subdir` for multi-session file writes.
  - Config/schema changes: keep `src/config-schema.ts` in sync with `octopi.schema.json` / `octopi.example.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JK-Ai-Era/octopi](https://github.com/JK-Ai-Era/octopi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
