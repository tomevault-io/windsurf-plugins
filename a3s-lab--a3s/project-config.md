---
trigger: always_on
description: This file provides repository guidance for Codex and other coding agents working in this repository.
---

# AGENTS.md

This file provides repository guidance for Codex and other coding agents working in this repository.

## Repository Shape

`a3s/` is a monorepo root, not a Rust workspace.

```text
a3s/
├── apps/
│   └── docs/         # Documentation site
├── crates/           # Rust crates as submodules
│   ├── acl/ ahp/ box/ code/ event/ gateway/
│   ├── lane/ memory/ power/ search/ updater/
│   └── common/       # Shared types (local crate)
└── homebrew-tap/     # Homebrew tap
```

Root rules:

- Do not create `Cargo.toml`, `src/`, or a Rust workspace in the repository root.
- Do not run `cargo init` or `cargo new` in the repository root.
- The root `justfile` is for monorepo orchestration only; do not treat root as a Rust crate.
- Work inside the relevant submodule or application directory.
- The root git remote should point to `git@github.com:A3S-Lab/a3s.git`.

## Git Safety

- Be careful with `git stash pop`; failed merge or rebase flows can destroy changes.
- Before `git pull --rebase`, commit or use `git stash push -m "<message>"`.
- After `git stash pop`, verify with `git diff --stat HEAD`.
- If stash pop fails, prefer `git stash branch recovery`.
- Never revert user changes unless explicitly asked.

## Adding A New Crate

New Rust crates are external repositories added as submodules.

1. Create the GitHub repository under `A3S-Lab` using PascalCase.
2. Initialize in `/tmp`, push, then add as a submodule under `crates/<name>`.
3. Commit `.gitmodules` changes in the root repository.
4. Update `README.md` modules, roadmap, and repository structure.
5. Never initialize a new Rust crate directly in the monorepo root.

Conventions:

| Item | Convention |
| --- | --- |
| GitHub repo | PascalCase, for example `MyNewCrate` |
| Submodule path | kebab-case, for example `crates/my-new-crate` |
| Rust crate name | `a3s-my-new-crate` |

## General Engineering Rules

- Search before implementing. Use `rg` first.
- Read affected files before modifying them.
- Challenge assumptions: check whether functionality already exists and whether the current layer should own the change.
- Prefer boring, explicit code over clever abstractions.
- Keep changes scoped to the user request.
- Do not future-proof without an actual need.
- Use a single source of truth; remove redundant wrappers and orphaned exports.
- Public Rust types should be `Send + Sync` where applicable.
- Avoid production panics; return contextual errors.
- All code and documentation must be in English.

## Rust Guidelines

- Follow Microsoft Rust Guidelines.
- Use Tokio for I/O; do not block inside async contexts.
- Use `BoxError` from `core/src/error.rs` where available and include useful context.
- Event keys are dot-separated lowercase: `<domain>.<subject>.<action>`.
- Run formatting before completion: `cargo fmt --all`.
- Run focused tests/checks from the crate workspace, not from the monorepo root.
- Prefer `cargo test -p <crate>` for focused validation; use crate-local `just` recipes only when that project defines them.

## Python SDK Guidelines

- Use `async` / `await`.
- Use `async with` for cleanup.
- Add type hints.

## Configuration

- Prefer HCL over TOML for product configuration.
- Use `.hcl` by default for new config files unless an existing local convention says otherwise.

## NestJS And DDD Rules

Every business module must follow the four-layer DDD structure:

```text
modules/{module-name}/
├── domain/
│   ├── entities/
│   ├── value-objects/
│   ├── repositories/
│   ├── services/
│   └── events/
├── application/
│   ├── commands/{feature}/
│   └── queries/{feature}/
├── infrastructure/
│   └── persistence/
├── presentation/
│   ├── controllers/
│   └── dto/
│       ├── request/
│       └── response/
└── {module}.module.ts
```

Critical rules:

- `domain/` is pure TypeScript. No NestJS or framework imports.
- `domain/services/` contains only `.interface.ts` files.
- Do not create `*.service.ts` re-export files in `domain/services/`.
- Repository interfaces live in `domain/repositories/` and use `I{Entity}Repository`.
- Repository implementations live in `infrastructure/persistence/`.
- Service interfaces use `I{Service}Service`; implementations live under `infrastructure/`.
- Commands and queries use the subdirectory pattern: `commands/{feature}/`.
- DTOs belong under `presentation/dto/request` or `presentation/dto/response`.
- Controllers are thin and delegate to `CommandBus` / `QueryBus`.
- Do not use `any` where a proper type can be expressed.
- Use constructor injection and avoid `new` inside injectable classes.
- Use `@nestjs/config` with validated environment variables.

## REST API Rules

All API responses use the `ApiResponseInterceptor` wrapper.

Success response:

```json
{
  "code": 200,
  "message": "Success",
  "data": {},
  "requestId": "uuid",
  "timestamp": "2026-04-17T00:00:00.000Z"
}
```

Error response:

```json
{
  "code": 404,
  "statusCode": "NOT_FOUND",
  "message": "Resource not found",
  "details": {},
  "requestId": "uuid",
  "timestamp": "2026-04-17T00:00:00.000Z"
}
```

Principles:

- `code` is the HTTP status code.
- `statusCode` is the business error code.
- Document errors with OpenAPI decorators.
- Keep response shape consistent.

## Database Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [A3S-Lab/a3s](https://github.com/A3S-Lab/a3s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
