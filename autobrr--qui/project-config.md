---
trigger: always_on
description: Repository guidelines for AI coding agents (Codex, Claude Code, etc.) working on qui.
---

# AGENTS.md

Repository guidelines for AI coding agents (Codex, Claude Code, etc.) working on qui.

## Owner Collaboration Notes

- Do not implement review-suggested or extra changes outside requested scope without explicit user approval first.
- Treat other agent/Codex/CodeRabbit feedback as input to discuss, not automatic action.

## Project Structure & Module Organization

The Go backend lives in `cmd/qui` (entrypoint) and `internal/` modules for configuration, qBittorrent, metrics, and API routing; shared helpers sit in `pkg/`. The React/Vite client is in `web/src` with static assets in `web/public`, and its production bundle must stay synced to `internal/web/dist`. Reference docs live under `docs/`, while Docker and compose files in the repository root support container workflows.

End-user docs live in the Docusaurus project under `documentation/docs/`. Prefer updating those for user-facing copy; `docs/` is mostly internal/engineering notes.

Keep `README.md` concise. Feature deep-dives belong in `documentation/docs/`, not the root README.

## Build, Test, and Development Commands

```bash
# Build
make build              # Frontend bundle + Go binary with version metadata
make backend            # Go binary only
make frontend           # Frontend bundle only

# Development
make dev                # Starts air (hot-reload) + pnpm dev
make dev-backend        # Backend only with hot-reload
make dev-frontend       # Frontend only

# Testing
make test               # go test -race -count=1 -v ./...
make test-openapi       # Validate OpenAPI spec after touching internal/web/swagger

# Linting
make lint               # Changed files only (fast, use during iteration)
make lint-json          # JSON output to lint-report.json

# Pre-commit
make precommit          # fmt + gofix + lint on changed files

# Formatting
make fmt                # gofmt + frontend eslint --fix on changed files
make gofix-changed      # Apply go fix on changed Go files only
make gofix-check-changed # Check go fix drift on changed Go files only
```

## Linting Strategy

The project uses golangci-lint v2 with strict configuration targeting AI-generated code patterns:

| Linter | Purpose | Threshold |
|--------|---------|-----------|
| dupl | Catch code duplication | 100 tokens |
| gocognit | Cognitive complexity | 15 |
| funlen | Function length | 80 lines |
| interfacebloat | Interface size | 5 methods |
| errcheck | Unchecked errors | All, including type assertions |
| gocritic | Non-idiomatic patterns | diagnostic + style + performance |

**Workflow:**
1. During implementation: `make precommit` (changed files only, fast feedback)
2. To fix issues: `make lint-fix` then address remaining manually

**Guardrail (web formatting):** avoid repo-wide `pnpm format` / `eslint --fix` sweeps unless explicitly requested. Prefer fixing only the files reported by lint for the current task/PR.

## Coding Style & Naming Conventions

Keep Go code `gofmt`-clean with PascalCase exports, camelCase locals, and package-level interfaces grouped by domain inside `internal/<area>`. The frontend follows ESLint @stylistic defaults: two-space indentation, double quotes, trailing commas on multiline literals, and Unix line endings. Organize React modules by feature within `web/src/{pages,routes,components}` and choose descriptive file names (e.g., `torrent-table.tsx`).

**Critical conventions:**
- Prefer explicit error handling over silent failures
- Keep interfaces small (≤5 methods)
- Avoid `map[string]interface{}` — use proper structs
- No backward compatibility shims unless explicitly requested
- **Loop variables (Go 1.22+):** Don't use `tt := tt` in parallel subtests — Go 1.22+ creates a new variable per iteration, so the old workaround is unnecessary and flagged by linters

**Single-user self-hosted context:** qui runs on someone's home server, not as a multi-tenant SaaS with untrusted input and complex failure modes. Skip paranoid defensive programming for impossible or purely theoretical scenarios. Code that guards against states that can't happen adds complexity without value. Prioritize readable, maintainable code over excessive robustness.

## Code Shape

- Prefer behavior-bearing branches only. If multiple `switch` cases return the same value as `default`, collapse them.
- In boolean classifiers, list only the exceptional cases (`true` cases or error cases). Let `default` handle the common path.
- Do not add documentation-only branches unless they enforce something mechanically via compiler, linter, or tests.
- When a branch only enumerates known states, ask whether it changes behavior, improves safety, or provides exhaustiveness checking. If not, delete it.

## React Effects

- Use `useEffect` only to sync with external systems (DOM, subscriptions, network).
- Avoid derived state in Effects; calculate during render, or `useMemo` for expensive compute.
- Put user-driven logic in event handlers, not Effects.
- To reset state, prefer a `key` or render-time adjustments instead of Effects.
- Fetch Effects must guard against stale responses (cleanup/abort).
- Source: https://react.dev/learn/you-might-not-need-an-effect

## Testing Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autobrr/qui](https://github.com/autobrr/qui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
