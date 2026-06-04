---
trigger: always_on
description: This file provides shared guidance to coding agents working in this repository. Claude Code reads the same guidance through the root `CLAUDE.md` stub, which contains `@AGENTS.md`.
---

# AGENTS.md

This file provides shared guidance to coding agents working in this repository. Claude Code reads the same guidance through the root `CLAUDE.md` stub, which contains `@AGENTS.md`.

**IMPORTANT**: This file must be written entirely in English. Do not use Japanese or any other languages in AGENTS.md.

## AGENTS.md Update Rules

**CRITICAL RULE**: AGENTS.md should only contain information that would cause problems if skipped. All other details belong in specialized documentation files.

### What belongs in AGENTS.md
- Critical requirements (e.g., `make check` before push)
- Essential commands for daily development
- Brief architecture overview with links to details
- Rules that, if skipped, cause real damage (data loss, broken builds, security issues)

### What does NOT belong in AGENTS.md
- Detailed implementation patterns → `dev-docs/patterns/`
- Comprehensive architecture details → `dev-docs/architecture-guide.md`
- Development workflow details → `dev-docs/development-insights.md`
- Issue management procedures → `dev-docs/issue-management.md`
- Testing best practices → `dev-docs/patterns/testing.md`

## Project Overview

spanner-mycli is a personal fork of spanner-cli, an interactive CLI for Google Cloud Spanner. Philosophy: "by me, for me", ZeroVer (never reaching v1.0.0), experimental features welcome.

**Terminology**: OSS spanner-cli, Google Cloud Spanner CLI (`gcloud alpha spanner cli`), spannercli (`spannercli sql`), and spanner-mycli (this project) are all distinct. Be specific when comparing.

## CRITICAL REQUIREMENTS

**Before ANY push to the repository**:
1. **Always run `make check`** - runs test && lint && fmt-check
2. **Resolve conflicts with origin/main** - ensure branch can merge cleanly
3. **Never push/commit directly to main branch** - always use feature branches + PRs
4. **Squash merge only** - enforced via Repository Ruleset
5. **PR merge process**: Use `go tool gh-helper reviews wait` before merging. Gemini review and summary are auto-triggered on PR creation — do NOT use `--request-review` or `--request-summary` at that point. After additional commits: use `--request-review` to get a new review. Use `--request-summary` only right before merge to update the summary.
6. **Squash merge commits**: MUST include descriptive summary of PR changes
7. **GitHub comment editing**: NEVER use `gh pr comment --edit-last` - always specify exact comment ID
8. **GitHub checks must pass**: All CI checks MUST pass before merging. Always investigate failures - never assume they are transient.

## Essential Commands

```bash
# Development cycle (CRITICAL)
make check                    # REQUIRED before ANY push (test + lint + fmt-check)
make build                    # Build the application
make test-quick               # Quick tests during development
make fmt                      # Format code

# Development tools (Go tool directive, managed via go.mod)
go tool gh-helper reviews fetch <PR>                    # Fetch review data
go tool gh-helper reviews fetch <PR> --unresolved-only  # Only unresolved threads
go tool gh-helper reviews wait <PR>                     # Wait for reviews + checks
go tool gh-helper threads reply <THREAD_ID> --commit-hash <HASH> --resolve
go tool gh-helper issues show <N> --include-sub         # Show issue with sub-issues
go tool gh-helper issues edit <N> --parent <P>          # Link as sub-issue
go tool gh-helper labels add bug,enhancement --items 254,267
go tool gh-helper releases analyze --milestone v0.19.0
go tool github-schema type <TypeName>                   # GraphQL schema introspection
```

For full gh-helper command reference, see [dev-docs/issue-management.md](dev-docs/issue-management.md).

## Core Architecture Overview

### Critical Components
- **main.go**: Thin entry point, calls `internal/mycli.Main()`
- **internal/mycli/app.go**: CLI argument parsing, configuration, `Main()` entry point
- **internal/mycli/session.go**: Database session management
- **internal/mycli/statements.go**: Core SQL statement processing
- **internal/mycli/system_variables.go**: System variable management
- **internal/mycli/client_side_statement_def.go**: **CRITICAL** - Defines all client-side statement patterns

### System Variable Conventions
- CLI-specific variables **MUST** use `CLI_` prefix
- All registrations in `internal/mycli/system_variables_registry.go`
- Details: [dev-docs/patterns/system-variables.md](dev-docs/patterns/system-variables.md)

### Regex Pattern Guidelines
- **Static patterns**: Precompile at package level (`var patternRe = regexp.MustCompile(...)`)
- **Dynamic patterns**: Compile at runtime, avoid caching unless profiling shows need

### Go Modernization Baseline
- This module targets Go 1.25. Use language features and standard-library APIs available through Go 1.25 when touching nearby code, while keeping modernization changes scoped.
- Do not add loop-variable shadow copies only for closure or parallel-subtest safety. Go 1.22+ gives loop variables per-iteration scope for modules declaring `go 1.22` or later.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apstndb/spanner-mycli](https://github.com/apstndb/spanner-mycli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
