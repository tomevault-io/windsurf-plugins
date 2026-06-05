---
trigger: always_on
description: Instructions for AI assistants working on this repository (Cursor, Claude Code, and compatible tools).
---

# APIHub Backend — Agent Instructions

Instructions for AI assistants working on this repository (Cursor, Claude Code, and compatible tools).

## Clarification before coding

- Do **not** generate or modify code until the task requirements are clear.
- Ask the user targeted questions when scope, behavior, acceptance criteria, or API contract is ambiguous.
- For GitHub ticket work, use the project skill `github-ticket-implementation-planner` before implementation.
- If you must assume something, state assumptions explicitly and keep changes minimal until confirmed.

## Error handling: fail fast, fix root cause (not symptoms)

Applies to **bug fixes and new features**.

### Bug fixes

- **Find and fix the root cause** — trace the failure (logs, stack, data flow, repro). Do not mask symptoms.
- **Forbidden as a “fix”** unless the user explicitly requests a temporary workaround and documents it:
  - Swallowing errors (`_ = err`, empty `catch`, `return nil` after failed I/O/DB/API calls).
  - Silent fallbacks to “default” behavior when an operation failed (empty result, zero value, skip step, pretend success).
  - Broad `recover()` or generic handlers that hide the real failure.
  - Replacing a returned error with a generic message without fixing why it failed.

### New code and refactors

- **Propagate errors** up the stack; return `error` from services/repositories; let controllers map to API error responses via `exception/ErrorCodes.go`.
- **Fail fast** when state is invalid or required setup failed (`log.Fatalf` in `Service.go` wiring, panic only where the codebase already does for unrecoverable programmer errors).
- **Log errors** at the appropriate layer (see `docs/development_guide.md` — errors to ERROR log); do not log-and-ignore.
- A **deliberate** fallback or default is allowed only when product requirements define it; document why in code or the ticket, and still log at WARN/ERROR when the primary path failed.

### Before submitting a bug-fix diff

Briefly state: **root cause**, **why the change fixes it**, and confirm you did **not** add swallow-and-continue logic.

## Libraries and dependencies

- Do **not** reimplement functionality that exists in well-established, industry-standard libraries.
- Search for suitable libraries before writing custom utilities (HTTP clients, parsing, crypto, etc.).
- Prefer dependencies already used in this repo; justify any new dependency briefly.

## GitHub CLI

- Use the **`gh`** CLI for GitHub issues, pull requests, checks, and releases.
- If `gh` is not installed or not authenticated, tell the user to install and authenticate it; do not rely on fragile HTML scraping or undocumented APIs.

## Cross-platform development (Windows + Linux)

- Team members use **Linux** and **Windows (often with WSL)**.
- Prefer **portable** commands: `bash` scripts with forward slashes, run from the **repository root**.
- On **Windows without WSL** in the active shell: use **WSL** (`wsl bash .cursor/skills/.../script.sh`), **Git Bash**, or **PowerShell** (`powershell -File .cursor/skills/.../script.ps1`).
- Do not assume Unix-only tools beyond `git`, `go`, `gh`, and `bash` unless the user confirms they are available.
- Avoid OS-specific path separators in instructions; use repo-relative paths like `qubership-apihub-service/...`.

## Related repositories (Helm, E2E tests)

Charts and Postman E2E collections live in **other repos** (not cloned in this workspace by default). When a feature needs them, **remind** the developer with links and concrete follow-ups — see [`docs/agent/related-repositories.md`](docs/agent/related-repositories.md).

Do not silently skip: after REST, config, or env changes, check that doc’s “remind when” lists and include a short **Related repositories** subsection in your completion summary.

## Repository layout (minimal orientation)

| Area | Location |
|------|----------|
| Main service entry / DI wiring | `qubership-apihub-service/Service.go` |
| HTTP controllers | `qubership-apihub-service/controller/` |
| Business logic | `qubership-apihub-service/service/` |
| Data access | `qubership-apihub-service/repository/` |
| DB entities + simple converters | `qubership-apihub-service/entity/` |
| API DTOs / views | `qubership-apihub-service/view/` |
| API error codes | `qubership-apihub-service/exception/ErrorCodes.go` |
| SQL migrations | `qubership-apihub-service/resources/migrations/` |
| OpenAPI specs | `docs/api/` (e.g. `APIHUB_API.yaml`, `Admin API.yaml`, `APIHUB_API_internal.yaml`) |
| Human docs index | `docs/README.md` |
| Development guide (logging, API-first, deprecation) | `docs/development_guide.md` |

## Go coding conventions (summary)

Detailed rules apply via `.cursor/rules/` and `.claude/rules/` when matching files are in context. Key points:

- **No magic numbers** — use named constants; if a literal is unavoidable, add a short comment explaining why.
- **Config defaults** — define once in `SystemInfoService.setDefaults()`; validate ranges in `config/Config.go` with `validate` tags; do not duplicate viper defaults as service-layer fallback constants.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Netcracker/qubership-apihub-backend](https://github.com/Netcracker/qubership-apihub-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
