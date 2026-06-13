---
trigger: always_on
description: This file captures conventions for AI agents and contributors working on [depx](https://github.com/projectdiscovery/depx).
---

# AGENTS.md — depx development guide

This file captures conventions for AI agents and contributors working on [depx](https://github.com/projectdiscovery/depx).

## Project overview

`depx` is a Go CLI for supply-chain attack intelligence: compromised-package feed, package/advisory lookups, local lockfile/SBOM scanning, and GitHub repository scanning via dependency-graph SBOM exports.

## Commands (user-facing)

| Command | Purpose |
|---------|---------|
| `depx` (default) | Recent compromised packages feed |
| `depx <ref>` / stdin | Check package refs or advisory IDs |
| `depx audit [path...]` | **Local** lockfile/SBOM audit only |
| `depx search <query>` | Search malicious packages by name |
| `depx github [target...]` | **GitHub-only** repo/org scanning |
| `depx id <MAL\|GHSA\|...>` | Advisory lookup |
| `depx version` / `depx --version` / `depx update` | Version info and self-update |

**Do not** add GitHub repo auditing to `depx audit`. GitHub targets belong exclusively on `depx github`. If audit receives a `github:` ref without a configured client, return a clear error pointing users to `depx github`.

## Code layout

```
cmd/depx/           main entry
internal/cli/       Cobra commands (split by concern)
  root.go           root command, flags, init
  feed.go           default feed
  check.go          implicit package checks
  id.go             advisory ID lookup
  audit_run.go      shared audit wiring (buildAuditOptions, runAudit)
  github.go         github subcommand
internal/lockfile/  canonical root lockfile names + ecosystem mapping
internal/github/    GitHub API client, parsing, SBOM/lockfile fetch
internal/audit/      local discovery, extraction, malicious-package matching
internal/intel/     Provider interface, backed by the local inventory index
internal/inventory/  inventory export model + streaming gzip fetch
internal/malindex/   in-memory malicious-package index (search + match + lookup)
internal/sync/    manifest-tracked hourly inventory sync (keeps last good index on failure)
internal/output/    JSON + human card rendering
e2e/                CLI integration tests
```

Keep `internal/cli/root.go` thin — add new command logic in dedicated files, not by growing root.go.

## Single sources of truth

### Root lockfiles

All root lockfile names live in **`internal/lockfile/lockfile.go`** (`RootNames`, `IsRootName`, `Ecosystem`).

- `internal/audit/discover.go` — local discovery
- `internal/github/lockfiles.go` — GitHub Contents API fetch
- `internal/audit/audit.go` — `inferEcoFromPath`

**Never duplicate** lockfile name lists elsewhere.

### GitHub target parsing

All GitHub URL/owner/repo parsing goes through **`internal/github/repo.go`**:

- `ParseTarget` — full parse (repo or org)
- `ParseRepo` — single repo only (wraps `ParseTarget`)
- `Repo.URL()` — canonical `https://github.com/owner/repo`

Do not reimplement URL parsing in CLI or audit packages.

### GitHub API errors

Use **`github.APIError`** and `errors.As` / `AsAPIError` — avoid string-matching `"status 404"` in new code.

### Audit options

CLI audit wiring is centralized in **`internal/cli/audit_run.go`**:

- `buildAuditOptions(ghClient)` — shared Options construction
- `runAudit(cmd, paths, ghClient)` — nil client for local audit; real client for `depx github`

Both `newAuditCmd` and `runGitHub` must use these helpers.

### Limit flags (`-n`)

`-n` / `--limit` semantics differ by command:

- **Default feed** (`depx -n N`): caps feed entries (`config.NormalizeFeedLimit`)
- **`depx search -n N`**: caps search results shown (default from config feed limit; footer shows total matches)
- **`depx github -n N`**: caps repos when target is org/user (`config.NormalizeGitHubRepoLimit`). Default depends on auth: `DefaultGitHubRepoLimit` (100) with a token, `DefaultGitHubRepoLimitUnauth` (10) without, to stay within GitHub's ~60 req/hr unauthenticated budget.

Both use shared `config.normalizeLimit` — extend that helper rather than duplicating validation.

## Output cards

Malicious advisory cards share a header via **`writeMaliciousCardHeader`** and body via **`writeFeedCard`**. Feed, search, malicious check, and `id` lookup should all route through those helpers; audit findings add context lines via **`writeAuditFindingCard`**.

## E2E tests

- **`e2e/harness_test.go`**: `TestMain` builds the binary once; use `binPath(t)`.
- **`e2e/mock_source_test.go`**: gzipped inventory export mock + JSON assertions (`mockSourceServer`, `sourceEnv`).
- **`e2e/mock_github_test.go`**: GitHub break-test mock server.

Do not rebuild the binary per test or duplicate mock servers across files.

Break/adversarial tests belong in `e2e/break_test.go` and `e2e/break_github_test.go`.

## Change discipline

1. **Minimal diffs** — match existing style; no drive-by refactors.
2. **No shell completion subcommand** — Cobra default completion is disabled.
3. **No `depx check` subcommand** — checks are `depx <ref>` or stdin; `depx check <ref>` is an optional alias (keyword stripped at root).
4. **Errors** — use `internal/apperr` (`Usage`, `Upstream`); wrapped causes must appear in `Error()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectdiscovery/depx](https://github.com/projectdiscovery/depx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
