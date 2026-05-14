---
trigger: always_on
description: npm run canaries         # or: make qa-quick
---

# Project Instructions

## Commands

```bash
# Typecheck
npx tsc --noEmit

# Run canaries (local-only; external report push has been removed)
npm run canaries         # or: make qa-quick

# QA review (orchestrated engine)
make qa-review REPO=/path/to/target                          # git-backed target
make qa-review REPO=/path/to/target ACCEPT_NO_GIT=1          # non-git target (prompt/flag required)

# Deltas and cleanup from local qa-data
npm run qa:delta         # markdown delta report
npm run qa:evolve-v2     # local-data evolution (replaces qa-evolve)
npm run qa:cleanup       # prune old runs

# Test credential profiles (stored in OS keychain)
make qa-creds-list                    # list saved profiles
make qa-creds-store NAME=staging-admin # store a new profile
make qa-creds-show NAME=staging-admin  # show profile details
make qa-creds-delete NAME=staging-admin # delete a profile

# Provider API keys + Gemini CLI auth (stored in OS keychain)
make qa-keys-check       # list keys already in the keychain
make qa-keys-setup       # print platform-specific add/update/delete commands

# Setup
make qa-setup            # npm ci
```

## Credential storage (OS keychain)

Service name is always `sparfuchs-qa`. Account names are the credential identifiers below.

| Account | Used by |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API provider (orchestrator) |
| `OPENAI_API_KEY` | OpenAI API provider |
| `XAI_API_KEY` | xAI API provider |
| `GOOGLE_GENERATIVE_AI_API_KEY` | Google Generative AI provider (Gemini API) |
| `GEMINI_API_KEY` | Gemini **CLI** auth (adapter injects it into the CLI's env at spawn) |

**macOS — add / update / delete / verify:**
```bash
# Add (or run to overwrite an existing entry after deleting)
security add-generic-password -s sparfuchs-qa -a GEMINI_API_KEY -w 'AIza...'

# Update (delete + re-add)
security delete-generic-password -s sparfuchs-qa -a GEMINI_API_KEY
security add-generic-password    -s sparfuchs-qa -a GEMINI_API_KEY -w 'new-value'

# Verify (prints the stored value to stdout — for confirmation only)
security find-generic-password   -s sparfuchs-qa -a GEMINI_API_KEY -w
```

**Linux (libsecret / secret-tool):**
```bash
echo 'AIza...' | secret-tool store --label=sparfuchs-qa service sparfuchs-qa key GEMINI_API_KEY
secret-tool lookup service sparfuchs-qa key GEMINI_API_KEY   # verify
secret-tool clear  service sparfuchs-qa key GEMINI_API_KEY   # delete
```

**Windows (PowerShell, CredentialManager module):**
```powershell
$s = ConvertTo-SecureString 'AIza...' -AsPlainText -Force
New-StoredCredential -Target 'sparfuchs-qa-GEMINI_API_KEY' -Password $s -Type Generic -Persist LocalMachine
```

**Fallbacks (not persisted across shells):**
```bash
export GEMINI_API_KEY=AIza...
```

**Gemini CLI — alternative one-time OAuth instead of an API key:**
```bash
gemini   # complete browser login once; creds cached at ~/.gemini/oauth_creds.json
```

Resolution priority (highest first): keychain → shell env var → `~/.gemini/oauth_creds.json` (Gemini CLI only). `make qa-keys-check` reports which keychain entries exist.

## Run mode vs. coverage strategy

Two independent axes. Both default to sensible values but do not mean the same thing — `mode: full` does NOT audit every file.

**`mode`** — which agents run
- `full` — every agent in the registry (default for `make qa-review`)
- `review` / `tier1` / `tier2` / `diff` / `selective` / `training` / `docs` — subsets

**`coverageStrategy`** — how deep file sampling goes
- `sweep` — ~40% target, fast
- `balanced` — ~65% target, default for non-full modes
- `thorough` — ~85% target, auto-default when `mode: full` (upgraded from `balanced` in Phase 4 of preflight-census work)
- `exhaustive` — ~100% target, audits every file

Override via `QA_COVERAGE_STRATEGY=exhaustive` or the `coverageStrategy:` field in `config/models.yaml`. Preflight prints the effective strategy + computed success criteria before dispatch — see `Preflight gate` below.

## Preflight gate

Every run prints a repo census, a plan preview, and scale-adaptive success criteria **before** dispatching agents. The operator confirms with `[y/N]`. When prior-run gaps are detected, a second prompt offers:
- `[1] Auto-heal` — inject heal jobs alongside the main wave
- `[2] Report only` — proceed; gaps surface in qa-gaps.md § Run Quality Deficit
- `[3] Fail + script` — abort, write `remediation-commands.sh` with targeted `AGENT_ONLY=<name>` / `MODULE=<path>` invocations

**Env bypasses:**
- `QA_PREFLIGHT=skip` — auto-accept everything (CI path; picks "Report only" for gap handling).
- `QA_GAP_HEAL=auto|report|fail` — pre-select gap-handling choice.
- `AGENT_ONLY=<agent-name>` — restrict the run to a single agent (used by the fail-script flow).
- `MODULE=<subpath>` — restrict discovery to one subdirectory.

Preflight writes `preflight.json` into the run dir; post-run, `run-verifier` grades actual results against it and writes `run-quality.json`. When any criterion misses, qa-report.md prepends a "Run Quality — PARTIAL" block and qa-gaps.md gets a "Run Quality Deficit" section with observed / expected / cause / remediation for each failing check. The release-gate verdict is NOT overridden.

## Architecture

- `canaries/` — QA canary checks (code-quality, security, perf, i18n, rbac)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sparfuchs-Corporation/sparfuchs-qa](https://github.com/Sparfuchs-Corporation/sparfuchs-qa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
