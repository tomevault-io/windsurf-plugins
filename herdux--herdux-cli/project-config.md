---
trigger: always_on
description: This file defines **mandatory rules** for AI agents and automated tools working in this repository.
---

# AGENTS.md — Herdux CLI

This file defines **mandatory rules** for AI agents and automated tools working in this repository.
Violating these rules is considered a bug.

Herdux is a deterministic, engine-agnostic CLI.
Clarity, separation of concerns, and explicit behavior are non-negotiable.

---

## QUICK REFERENCE

| Task                      | Workflow                           |
| ------------------------- | ---------------------------------- |
| Add a new database engine | `.agents/workflows/new-engine.md`  |
| Add a new CLI command     | `.agents/workflows/new-command.md` |
| Refactor existing code    | `.agents/workflows/refactoring.md` |
| Write or review tests     | `.agents/workflows/testing.md`     |
| Commit and open a PR      | `.agents/workflows/pre-commit.md`  |

---

## ROLE & EXPECTATIONS

- Act as a **senior CLI / Node.js engineer**
- Prefer **explicit, boring, predictable code**
- Optimize for **maintainability over cleverness**
- Never introduce hidden behavior or magic defaults
- When in doubt, **fail loudly**

---

## ARCHITECTURE OVERVIEW (MANDATORY)

Herdux follows a strict layered architecture.
**Layer boundaries MUST NOT be violated.**

```
src/
├── index.ts        # CLI entrypoint (flags & command registration)
├── commands/       # WHAT to do (engine-agnostic verbs)
├── core/           # Pure contracts (interfaces only)
├── infra/          # HOW to do it (engines, config, binaries)
└── presentation/   # HOW to display (logging & output)
```

---

### core/ — Contracts (Gravitational Center)

- Contains **only interfaces and types**
- Defines `IDatabaseEngine`
- **MUST NOT** import infra, commands, or presentation
- **MUST NOT** reference PostgreSQL, MySQL, Docker, or binaries
- **MUST NOT** contain logic

If something depends on everything else, it belongs here.

---

### infra/ — Implementations

- Contains **all concrete behavior**
- Owns:
  - engine implementations (Postgres, MySQL, etc.)
  - binary execution (`psql`, `mysql`, `pg_dump`, `mysqldump`)
  - config resolution (`~/.herdux/config.json`)
  - environment validation and auto-discovery

Rules:

- **NEVER** execute external binaries outside `infra/`
- **NEVER** bypass `command-runner.ts`
- **NEVER** assume a default engine
- **NEVER** leak engine-specific details to commands

---

### commands/ — CLI Verbs

- Define **user-facing commands** only
- Are **100% engine-agnostic**
- Always resolve engines via `engine-factory`
- Always call:
  - `checkClientVersion()`
  - methods defined in `IDatabaseEngine`

Rules:

- **MUST NOT** call binaries
- **MUST NOT** read config files directly
- **MUST NOT** reference `psql`, `mysql`, or engine internals
- **MUST NOT** contain connection-resolution logic

Commands decide _what_, never _how_.

---

### presentation/ — Output

- Responsible only for formatting and logging
- **MUST NOT** contain business logic
- **MUST NOT** affect control flow

---

## COMMAND FLOW (REFERENCE MODEL)

Example: `hdx --engine mysql list`

```
index.ts
└── commands/list.ts
    └── resolveEngineAndConnection()   # resolve-connection.ts
        └── engine-factory.ts → createEngine("mysql")
    └── mysql.engine.ts
        └── command-runner.ts          # execa wrapper
```

This flow is **canonical**. Do not invent alternatives.

---

## SUPPORTED ENGINES

| Engine     | EngineType   | Default Port | Required Binaries               | Backup Formats                   |
| ---------- | ------------ | ------------ | ------------------------------- | -------------------------------- |
| PostgreSQL | `"postgres"` | 5432         | `psql`, `pg_dump`, `pg_restore` | custom (`.dump`), plain (`.sql`) |
| MySQL      | `"mysql"`    | 3306         | `mysql`, `mysqldump`            | plain (`.sql`)                   |
| SQLite     | `"sqlite"`   | N/A          | `sqlite3`                       | custom (`.db`), plain (`.sql`)   |

To add a new engine, follow `.agents/workflows/new-engine.md`.

---

## ENGINE RULES

- Engines implement `IDatabaseEngine`
- Engines:
  - validate required binaries
  - execute commands via `command-runner`
  - return structured results (`stdout`, `stderr`, `exitCode`)
- Engines **MUST NOT**:
  - print directly to stdout
  - prompt the user
  - read CLI flags
  - assume interactive mode

---

## command-runner.ts (CRITICAL)

- Thin wrapper over `execa`
- Standardizes:
  - timeouts
  - env
  - stdin (used for restore)
  - return shape: `{ stdout, stderr, exitCode }`

Rules:

- **ALWAYS** use `command-runner`
- **NEVER** call `execa` directly elsewhere
- **NEVER** change its return contract casually

---

## CONNECTION RESOLUTION

`resolve-connection.ts` owns all connection logic.

Order of precedence:

1. Explicit CLI flags (`--host`, `--port`, `--user`, `--password`)
2. Saved server profiles (`-s <name>`)
3. Config defaults (`hdx config set`)
4. Auto-discovery (port scanning — last resort)

Rules:

- **NEVER** duplicate connection logic
- **NEVER** assume host or port defaults in commands
- **NEVER** skip resolution
- In tests, set `HERDUX_TEST_FORCE_TTY=1` to simulate TTY without a real terminal

---

## infra/docker/ — TEST INFRASTRUCTURE ONLY

This directory contains Docker Compose files for **E2E testing only**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [herdux/herdux-cli](https://github.com/herdux/herdux-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
