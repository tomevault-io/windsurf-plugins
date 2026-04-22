---
trigger: always_on
description: This document provides context for AI agents working on the Tokscale project.
---

# AI Agent Guidelines

This document provides context for AI agents working on the Tokscale project.

## Maintaining AGENTS.md Files

When updating AGENTS.md files, follow these principles:

- **No hardcoded counts** — Don't write "10 crates" or "5 modules"; these become outdated instantly
- **No exhaustive lists** — Prefer dynamic commands (`ls crates/`) over maintaining complete lists
- **Document constraints, not descriptions** — Focus on non-obvious behaviors, gotchas, and cross-crate dependencies
- **Use nested AGENTS.md** — Place crate-specific details in `crates/{name}/AGENTS.md`, not here
- **Verify before documenting** — Grep/read the code to confirm claims are accurate
- **Delete outdated info** — Outdated docs are worse than no docs

## Commit Message Convention

```
<type>: <description>

[optional body]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code refactoring (no behavior change) |
| `docs` | Documentation only |
| `test` | Adding or updating tests |
| `chore` | Maintenance tasks |
| `perf` | Performance improvements |

### Examples

```
feat: add session branching with /fork command
fix: handle empty response from provider
refactor: extract streaming logic to separate module
docs: update README with new CLI options
```

### Commit Message & PR Title Rules (CRITICAL)

> These rules apply to **both commit messages AND pull request titles**. PR titles become the squash-merge commit message, so they must follow the same conventions.

**DO:**
- Describe the actual change in plain, technical terms
- Keep commits atomic (one logical change per commit)
- Use the format: `<type>(<scope>): <what changed and why>`

**DON'T:**
- Reference internal review labels (P0, P1, P2, etc.) in commits or PR titles
- Mention "Oracle", "audit", "review findings", "hardening" in commits or PR titles
- Use agent-internal jargon: "wave", "hardening", "compliance", "verification pass"
- Bundle multiple unrelated fixes into one commit
- Use vague messages like "fix issues" or "address feedback"

**Good Examples:**
```
fix(lsp): pass server args to stdio spawn command
fix(lsp): convert 1-indexed input lines to 0-indexed LSP positions
fix(gemini): parse SSE data frames instead of raw JSON lines
fix(orchestrator): route provider tools through approval flow
```

**Bad Examples (NEVER do this):**
```
fix: address P0 issues from Oracle review      ❌
fix(hardening): Oracle Round 4 fixes           ❌
fix: audit findings                            ❌
fix: various improvements                      ❌
fix(tui): harden unreleased changes — P0-P3    ❌  (PR title)
fix: hardening wave 1 compliance fixes         ❌  (PR title)
```

## Agent Command Execution

- When running `tokscale` CLI commands from an automated agent (tests, CI, or tool-driven shells), always pass `--no-spinner` unless spinner behavior is the thing being tested.
- This avoids non-interactive terminal issues and keeps command output stable for assertions and logs.

## Release & Deployment

### Overview

Releases are published to npm via a GitHub Actions `workflow_dispatch` pipeline, followed by a manually created GitHub Release with handwritten notes. There is no staging environment — publishes go directly to npm `latest`.

### Release Pipeline

**Workflow:** `.github/workflows/publish-cli.yml`

**Trigger:** Manual — GitHub Actions UI → "Publish" → "Run workflow"

**Inputs:**
- `bump`: Version bump type — `patch (x.x.X)` | `minor (x.X.0)` | `major (X.0.0)`
- `version` (optional): Override string (e.g., `2.0.0-beta.1`), takes precedence over bump

**Stages (sequential):**

| # | Job | Description |
|---|-----|-------------|
| 1 | `bump-versions` | Reads current version from `packages/cli/package.json`, calculates new version, updates the Rust workspace version plus the CLI, wrapper, and platform package manifests, then uploads the bumped manifests as an artifact |
| 2 | `build-cli-binary` | 8-target parallel native Rust builds (macOS x86/arm64, Linux glibc/musl x86/arm64, Windows x86/arm64) |
| 3 | `publish-platform-packages` | Publishes platform-specific packages (`@tokscale/cli-darwin-arm64`, etc.) containing native binaries to npm |
| 4 | `publish-cli` | Publishes `@tokscale/cli` to npm (binary dispatcher + optionalDependencies) |
| 5 | `publish-alias` | Publishes `tokscale` wrapper package to npm |
| 6 | `finalize` | Commits the bumped release manifests back to repo as `chore: bump version to X.Y.Z` (authored by `github-actions[bot]`) |

**Duration:** ~15-20 minutes end-to-end.

**Package publish chain:** `@tokscale/cli` (with platform packages as optionalDependencies) → `tokscale` (depends on cli). Each waits for the previous to succeed.

### Post-Pipeline: Git Tag & GitHub Release

The CI pipeline does **NOT** create the git tag or GitHub Release. After the workflow completes successfully:

1. Verify the `chore: bump version to X.Y.Z` commit was pushed by CI
2. Create a GitHub Release manually:
   - **Tag:** `vX.Y.Z` (e.g., `v1.2.1`)
   - **Target:** The `chore: bump version to X.Y.Z` commit
   - **Title:** See [Release Notes Style](#release-notes-style) below
   - **Body:** See [Release Notes Template](#release-notes-template) below

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junhoyeo/tokscale](https://github.com/junhoyeo/tokscale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
