---
trigger: always_on
description: Every gh command, plus a local store, cross-org rollups, and FTS no other GitHub tool ships. Trigger phrases: `what's open across my org on github`, `cross-repo pr review load`, `github flake rank`, `release diff between tags`, `bottlenecks in github`, `use github-pp`, `run github-pp`.
---


# GitHub — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `github-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install github --cli-only
   ```
2. Verify: `github-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails (no Node, offline, etc.), fall back to a direct Go install (requires Go 1.26.3 or newer):

```bash
go install github.com/mvanhorn/printing-press-library/library/developer-tools/github/cmd/github-pp-cli@latest
```

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

github-pp-cli matches `gh` surface for surface and adds a sync engine + SQLite store so you can answer cross-repo questions in one query — `recent`, `bottlenecks`, `flake-rank`, `release-diff`, `reviewer-load`, `velocity`, `who-broke-main` — plus FTS that bypasses GitHub's 1000-result cap and a rate-limit-aware paginate that doesn't cliff-fail. Every command is agent-native by default: `--json`, `--select`, typed exit codes, `--dry-run` on every mutation.

## When to Use This CLI

Reach for github-pp-cli when the question is cross-repo or cross-org, when you need a structured + filtered answer (every command takes `--json --select`), when you want to grep beyond GitHub's 1000-result search cap, or when you want typed exit codes for orchestration. Use `gh` for one-shot interactive flows like `gh pr create` and `gh repo clone` — github-pp-cli wraps both surfaces but `gh`'s human ergonomics for those are world-class.

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Cross-org rollups
- **`reviewer-load`** — See review load (given, received, pending) per teammate over a window — for 1:1 prep, perf cycles, and cross-squad rollups.

  _Reach for this when you need to know who's overloaded with reviews — `gh` cannot answer this without N round trips, and the 1000-result search cap breaks it on big orgs._

  ```bash
  github-pp-cli reviewer-load --org acme --since 30d --json --select user,reviews_given,reviews_pending,median_ttr_hours
  ```
- **`recent`** — One command, all repos: PRs merged, issues closed, releases cut, runs failed since a watermark.

  _Reach for this for daily / weekly cross-repo standup rollups instead of opening N tabs of `gh` invocations._

  ```bash
  github-pp-cli recent --since 24h --org acme --json --select kind,repo,number,title,actor
  ```
- **`velocity`** — Per-repo throughput: PRs/day merged, median PR lifetime, p50 time-to-first-review, median CI duration.

  _Reach for this for engineering health checks and rollups instead of pasting `gh pr list --json` into a spreadsheet._

  ```bash
  github-pp-cli velocity --org acme --window 30d --json --select repo,merged_per_day,median_lifetime_h,p50_ttfr_h
  ```

### Local store leverage
- **`fts`** — Full-text search across every cached issue, PR, commit, and release across every synced repo — bypasses GitHub's 1000-result cap.

  _Use this when GitHub's search would silently truncate at 1000 hits or when you need to grep comments + bodies + commit messages in one shot._

  ```bash
  github-pp-cli fts "flaky test" --type prs,issues --since 90d --json --select repo,number,title,updated_at
  ```
- **`bottlenecks`** — Surfaces PRs open >N days awaiting review, PRs red >24h, issues without a triage label after 48h.

  _Use this for Friday triage or sprint hygiene — one command instead of six tabs._

  ```bash
  github-pp-cli bottlenecks --org acme --review-stale 7d --check-stale 24h --json --select kind,repo,number,reason,age_hours
  ```

### CI / release forensics
- **`flake-rank`** — Per-job success rate over the last N runs of a workflow, ranked by flakiness with mean duration and last failure.

  _Use this to triage 'is this job flaking for the team or just on my PR' — `gh run list` cannot rank._

  ```bash
  github-pp-cli flake-rank --workflow ci.yml --repo acme/web --since 50runs --json --select job,success_rate,mean_duration_s,last_failure
  ```
- **`release-diff`** — Show PRs merged, issues closed, contributors, and files changed between two tags in one repo.

  _Reach for this to draft release notes or audit what shipped between two tags without walking `git log` and pasting SHAs into `gh pr view`._

  ```bash
  github-pp-cli release-diff v2.14.0 v2.15.0 --repo acme/web --json --select prs,authors,files_changed
  ```
- **`who-broke-main`** — For each red main-branch run, the merge commit, PR, author, and elapsed-red duration.

  _Use this for incident retros and deploy postmortems instead of scrolling `gh run list` and clicking through._

  ```bash
  github-pp-cli who-broke-main acme/web --since 7d --json --select run_id,pr,author,started_at,red_duration_s
  ```

### Agent-native plumbing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cpard/github-pp-cli](https://github.com/cpard/github-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
