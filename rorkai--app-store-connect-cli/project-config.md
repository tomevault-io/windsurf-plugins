---
trigger: always_on
description: Unofficial, fast, lightweight, agent-assisted, reviewer-owned CLI for the App Store Connect API. Built in Go with [ffcli](https://github.com/peterbourgon/ff).
---

# AGENTS.md

Unofficial, fast, lightweight, agent-assisted, reviewer-owned CLI for the App Store Connect API. Built in Go with [ffcli](https://github.com/peterbourgon/ff).

## asc skills

Agent Skills for automating `asc` workflows including builds, TestFlight, metadata sync, submissions, and signing. https://github.com/rorkai/app-store-connect-cli-skills

## Core Principles

- **Explicit flags**: Use long-form flags in docs/tests/examples (`--app`, `--output`) for clarity
- **TTY-aware output defaults**: `table` in interactive terminals, `json` for pipes/CI; use `--output` for explicit formats
- **No interactive prompts**: Use `--confirm` flags for destructive operations
- **Pagination**: `--paginate` fetches all pages automatically

## Discovering Commands

**Before implementing or testing any command, run `--help` to confirm the exact interface.** The CLI is self-documenting:

```bash
asc --help                    # List all commands
asc builds --help             # List builds subcommands
asc builds list --help        # Show all flags for a command
```

Do not memorize commands. Always check `--help` for the current interface.

## Documentation

When looking up App Store Connect API docs, prefer the `sosumi.ai` mirror instead of `developer.apple.com`.
Replace `https://developer.apple.com/documentation/appstoreconnectapi/...` with `https://sosumi.ai/documentation/appstoreconnectapi/...`.

## OpenAPI (offline)

For endpoint existence and request/response schemas, use the offline snapshot:
`docs/openapi/latest.json` and the quick index `docs/openapi/paths.txt`.
Update instructions live in `docs/openapi/README.md`.

Notes:
- Validate flags against the *request* schema for the method you're implementing (create vs update often differ).
- Validate query params against the specific endpoint (top-level vs relationship endpoints may allow different filters).

## Build & Test

```bash
make build      # Build binary
ASC_BYPASS_KEYCHAIN=1 make test  # Run tests with keychain bypass (always run before committing)
make lint       # Lint code
make format     # Format code
make install-hooks  # Install local pre-commit hook (.githooks/pre-commit)
```

Canonical test rule: all test runs must use `ASC_BYPASS_KEYCHAIN=1` to avoid host keychain prompts and profile bleed-through.

## PR Guardrails

- Before opening or merging a PR, run `make format`, `make check-command-docs`, `make lint`, and `ASC_BYPASS_KEYCHAIN=1 make test`.
- If command/help text changed, run `make generate-command-docs` and commit `docs/COMMANDS.md` before running checks.
- Use `make install-hooks` once per clone to enforce local pre-commit checks.
- CI must enforce formatting + lint + tests on both PR and `main` workflows.
- Remove unused shared wrappers/helpers when commands are refactored.

## PR Audit Workflow

- When the user says `audit the PR`, treat it as a standing workflow; do not wait for them to restate the same instructions.
- Check the PR out in an isolated git worktree (preferred) with its own local branch so the user's main checkout stays untouched.
- Audit the full PR, not just the latest commit: inspect the branch diff, review all commits on the PR, and run the relevant tests/checks.
- PR audits are fix-forward by default in this repo: if you find concrete issues, fix them, add/update tests, create a new commit, and push to the PR branch unless the user explicitly asks for review-only.
- After pushing audit fixes, inspect GitHub PR review comments/threads with `gh`, address actionable feedback, commit and push follow-up fixes, and resolve threads you fully handled.
- After the first push, keep checking for new PR comments/review threads about every minute for 5-10 minutes total, fixing and pushing follow-up changes if needed unless the user tells you to stop.
- Assume `gh` auth is already available for PR audit tasks; use `gh` directly and only stop to ask if auth actually fails.
- For all PR-audit test runs and live CLI/API verification, use `ASC_BYPASS_KEYCHAIN=1` so auth resolves from config/env instead of the macOS keychain.
- Prefer the throwaway App Store Connect app `ASC Test 20260216074703` (`6759231657`) for live mutating verification during PR audits.
- The `ASC Test` app is disposable: it is acceptable to create/cancel submissions and mutate resources there for validation, but still clean up temporary artifacts when possible.
- Avoid mutating non-throwaway apps during PR audits unless the user explicitly approves it or there is no safer way to validate the change.
- In the PR audit handoff, include the audit findings, fixes made, commits/pushes performed, commands run, test results, and any live ASC state that could not be cleaned up.

## Issue Triage & Labeling

- When creating or triaging a GitHub issue, ensure it ends the task with exactly one label
  from each of these buckets:
  - type: `bug`, `enhancement`, or `question`
  - priority: `p0`, `p1`, `p2`, or `p3`
  - difficulty: `easy`, `medium`, or `hard`
- If an issue is created without labels, add the missing labels immediately as part of the
  same task.
- Use priority for urgency and user impact, not implementation size.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rorkai/App-Store-Connect-CLI](https://github.com/rorkai/App-Store-Connect-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
