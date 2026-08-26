---
trigger: always_on
description: Do NOT use worktrees. Make all changes directly on the current working branch.
---

# Charon — Claude Code Instructions

Do NOT use worktrees. Make all changes directly on the current working branch.

## Code Quality Guidelines

Every session should improve the codebase, not just add to it. Actively refactor code you encounter, even outside of your immediate task scope. Think about long-term maintainability and consistency. Make a detailed plan before writing code. Always create unit tests for new code coverage.

- **ARCHITECTURE AWARENESS**: Always consult `ARCHITECTURE.md` at the repository root before making significant changes to core components, system architecture, technology stack, deployment configuration, or directory structure.
- **DRY**: Consolidate duplicate patterns into reusable functions, types, or components after the second occurrence.
- **CLEAN**: Delete dead code immediately. Remove unused imports, variables, functions, types, commented code, and console logs.
- **LEVERAGE**: Use battle-tested packages over custom implementations.
- **READABLE**: Maintain comments and clear naming for complex logic. Favor clarity over cleverness.
- **CONVENTIONAL COMMITS**: Write commit messages using `feat:`, `fix:`, `chore:`, `refactor:`, or `docs:` prefixes.
- **`(security)` SCOPE**: For genuinely security-relevant `feat`/`fix` commits (real vulnerability fixes, new protective mechanisms — not general bug fixes), use `feat(security): <subject>` or `fix(security): <subject>`. This scope feeds a dedicated "Security" category in the What's New changelog, so it's reserved for real security work — overusing it for visibility on ordinary fixes dilutes the category's signal. **Vague by default**: the subject line must describe the *category* of issue and mitigation in general terms, and must NEVER reveal the specific vulnerability class, attack vector, or exact vulnerable code path — the changelog displays it verbatim to every self-hosted user, including ones running un-upgraded, still-vulnerable instances. Good: `fix(security): harden input validation in the API layer`. Bad: `fix(security): fix SQL injection in host search filter`.

## Governance & Precedence

When policy statements conflict across documentation sources:

1. **Highest Precedence**: This `CLAUDE.md` file (canonical source of truth for Claude Code)
2. **Agent Overrides**: `.claude/agents/**` files (agent-specific customizations)
3. **Operator Documentation**: `SECURITY.md`, `docs/security.md`, `docs/features/notifications.md`

**Reconciliation Rule**: When conflicts arise, the stricter security requirement wins.

## 🚨 CRITICAL ARCHITECTURE RULES 🚨

- **Single Frontend Source**: All frontend code MUST reside in `frontend/`. NEVER create `backend/frontend/` or any other nested frontend directory.
- **Single Backend Source**: All backend code MUST reside in `backend/`.
- **No Python**: This is a Go (Backend) + React/TypeScript (Frontend) project. Do not introduce Python scripts or requirements.

## 🛑 Root Cause Analysis Protocol (MANDATORY)

**Constraint:** You must NEVER patch a symptom without tracing the root cause.
If a bug is reported, do NOT stop at the first error message found. Trace the entire flow from frontend action to backend processing. Identify the true origin of the issue.

**The "Context First" Rule:**
Before proposing ANY code change or fix, build a mental map of the feature:
1. **Entry Point:** Where does the data enter? (API Route / UI Event)
2. **Transformation:** How is the data modified? (Handlers / Middleware)
3. **Persistence:** Where is it stored? (DB Models / Files)
4. **Exit Point:** How is it returned to the user?

**Anti-Pattern Warning:**
- Do not assume the error log is the *cause*; it is often just the *victim* of an upstream failure.
- If you find an error, search for "upstream callers" to see *why* that data was bad in the first place.

## Big Picture

- Charon is a self-hosted web app for managing reverse proxy host configurations with the novice user in mind. Everything should prioritize simplicity, usability, reliability, and security, all rolled into one simple binary + static assets deployment. No external dependencies.
- Users should feel like they have enterprise-level security and features with zero effort.
- `backend/cmd/api` loads config, opens SQLite, then hands off to `internal/server`.
- `internal/config` respects `CHARON_ENV`, `CHARON_HTTP_PORT`, `CHARON_DB_PATH` and creates the `data/` directory.
- `internal/server` mounts the built React app (via `attachFrontend`) whenever `frontend/dist` exists.
- Persistent types live in `internal/models`; GORM auto-migrates them.

## Backend Workflow

- **Run**: `cd backend && go run ./cmd/api`.
- **Test**: `go test ./...`.
- **Static Analysis (BLOCKING)**: Fast linters run automatically on every commit via lefthook pre-commit-phase hooks.
  - **Staticcheck errors MUST be fixed** — commits are BLOCKED until resolved
  - Manual run: `make lint-fast` or `make lint-staticcheck-only`
  - Full golangci-lint (all linters): Use `make lint-backend` before PR (manual stage)
- **API Response**: Handlers return structured errors using `gin.H{"error": "message"}`.
- **JSON Tags**: All struct fields exposed to the frontend MUST have explicit `json:"snake_case"` tags.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wikid82/Charon](https://github.com/Wikid82/Charon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
