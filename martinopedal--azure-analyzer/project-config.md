---
trigger: always_on
description: Bundle multiple Azure assessment tools into a single, portable runner. Output unified JSON + HTML/Markdown reports.
---

# Copilot Instructions — azure-analyzer

## Repository Purpose
Bundle multiple Azure assessment tools into a single, portable runner. Output unified JSON + HTML/Markdown reports.

## Query format
- ARG queries live in `queries/` as JSON (not .kql files)
- Every query MUST return a `compliant` column (boolean)
- See alz-graph-queries repo for query schema reference

## Branch protection
- Signed commits NOT required (breaks Dependabot and GitHub API commits)
- 0 required reviewers (solo-maintained)
- enforce_admins = true, linear history, no force push
- ✅ Required status checks: `Analyze (actions)` only (Python removed — repo is PowerShell)

## CodeQL policy
- This repo scans GitHub Actions workflows only — `language: [actions]`
- PowerShell is NOT scanned by CodeQL (no supported CodeQL extractor for PS)
- Actions scanning covers workflow injection risks (expression injection, untrusted input)

## SHA-pinning
- All GitHub Actions MUST use SHA-pinned versions, not tags
- Example: `actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6`

## Permissions
- Azure tools need Reader only — NO write permissions
- See PERMISSIONS.md for full breakdown per tool

## Documentation rules — ALWAYS required

Every PR that changes code, queries, or configuration MUST include a docs update in the same commit:

- ✅ `README.md` — update feature list, supported tools, permissions summary if changed
- ✅ `PERMISSIONS.md` — update if new Azure/Graph/GitHub API scopes are added
- ✅ `CHANGELOG.md` — add an entry for every user-visible change (feature, fix, breaking)
- ✅ Inline comments in new PowerShell modules if the logic is non-obvious

**No code PR merges without a matching docs update. This is not optional.**

## Issue conventions

- ✅ Every new issue MUST have the `squad` label — this is how Ralph (squad watch) picks it up for dispatch
- ✅ The auto-label-issues workflow adds `squad` automatically on open — never remove it
- ✅ Use labels `enhancement`, `bug`, `documentation` alongside `squad` to signal priority and type
- ✅ Issue titles must follow: `feat:`, `fix:`, `docs:`, `chore:` prefix

## Actions version policy
- Use SHA-pinned versions of actions/checkout (v6) and actions/setup-python (v6) — always pin by SHA, not tag

## GitHub-first principle
Validate changes in GitHub Actions, not locally. Push, trigger workflow, check logs, iterate.

## Agent session contract — always open a PR, always check history first

Every agent session MUST follow this contract. No exceptions for "small" changes.

### 1. Always open a PR (and surface the link)

- ✅ When a session produces **any** commit, the agent MUST call `create_pull_request` (draft is fine) before ending the session — even if the user did not explicitly ask for one. Pushing the branch via `report_progress` alone is **not sufficient**: a branch with no PR is invisible in the dashboard and triggers no CI / Copilot review.
- ✅ The final assistant reply MUST include the PR URL (`https://github.com/martinopedal/azure-analyzer/pull/<n>`) so the maintainer can click straight through. If `create_pull_request` reports an existing PR for the branch, surface that PR's URL instead.
- ✅ If the session is exploratory / informational (no commits, no branch changes), explicitly say so in the final reply — do not silently exit.
- ❌ Never end a session with "changes pushed" but no PR link. That is the primary failure mode this contract exists to prevent.

### 2. Always audit CI and check all context before doing work

Before writing code, editing files, or filing issues, the agent MUST complete **every** item below — in order. Skipping any step because the task feels small is a contract violation.

- ✅ **CI audit (first)** — use `list_workflow_runs` (GitHub MCP) to get the last 10 runs on `main` (and on the working branch if it exists). For any run whose `conclusion` is not `success`, use `get_job_logs` / `get_workflow_run_logs` to read the failed-job output. Name the failure in plain prose before touching code. A failing `main` pipeline means the repo is already broken — starting new work on top of it races with the fix and wastes CI credits.
- ✅ **Recent agent sessions** — read prior `report_progress` checklists / commit history on the working branch (`git log --oneline -20`) and on `origin/main` to avoid redoing or undoing recent work.
- ✅ **Squad memory** — review all stored memories tagged to this repo. Verify each relevant memory against the current code before relying on it (memories can be obsolete). If a memory no longer matches reality, overwrite it with `store_memory` before proceeding.
- ✅ **Open issues** — search for related `squad`-labelled issues so we don't duplicate filed work. Use `search_issues` / `list_issues` with the relevant keyword.
- ✅ **Open PRs** — `list_pull_requests state=open` to detect a sibling PR that already addresses the same area; coordinate or rebase rather than racing.
- ✅ **Copilot instructions** — re-read `.github/copilot-instructions.md` and `.copilot/copilot-instructions.md` at session start; conventions evolve and stale memory is not a defence.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinopedal/azure-analyzer](https://github.com/martinopedal/azure-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
