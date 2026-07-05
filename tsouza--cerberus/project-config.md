---
trigger: always_on
description: Drop-in **Prometheus / Loki / Tempo** HTTP gateway for **ClickHouse**. Parses each query language (PromQL with the upstream Apache prometheus parser; LogQL and TraceQL with cerberus's own in-house Apache reimplementations), lowers into a shared plan IR (`internal/chplan`), applies a small rule-based optimizer, and emits parameterised ClickHouse SQL. The HTTP layer speaks the upstream Prom / Loki / Tempo wire format so Grafana sees cerberus as three drop-in datasources.
---

# Cerberus — agent context

Drop-in **Prometheus / Loki / Tempo** HTTP gateway for **ClickHouse**. Parses each query language (PromQL with the upstream Apache prometheus parser; LogQL and TraceQL with cerberus's own in-house Apache reimplementations), lowers into a shared plan IR (`internal/chplan`), applies a small rule-based optimizer, and emits parameterised ClickHouse SQL. The HTTP layer speaks the upstream Prom / Loki / Tempo wire format so Grafana sees cerberus as three drop-in datasources.

## Hard rules (non-negotiable)

- **PR-per-change.** No direct pushes to `main` — branch protection rejects them. Required CI checks: `check`, `lint`, `forbid-skip`, `compatibility/{prometheus,loki,tempo}`, `probe`, `roundtrip (promql)`, `roundtrip (logql)`, `roundtrip (traceql)`, `compose-smoke`. The `dashboard` full-stack smoke (k3d + cerberus + Grafana + Playwright; lives as the `dashboard` job inside `.github/workflows/e2e.yml`) runs on push-to-main + nightly + manual dispatch only — it is informational on merges, not a PR gate. Force-push and deletion are off on `main`; the GitHub "Update branch" button (merge-commits) works for stale PRs. **Never use `gh pr merge --admin`** — every PR must merge cleanly with all required checks green. If a required check is failing, fix the code or fix the workflow; don't bypass. Branch protection has `enforce_admins: true` and the personal token doesn't grant override.
- **Agent-driven work goes through PRs, not Issues.** When *you* (an AI assistant) are doing the work, capture intent in the PR description — don't open an issue to track follow-up. Human contributors (or the maintainer) **are welcome to open issues** for bug reports, design discussions, feature proposals — the issues feature is on. The rule is about agent workflow hygiene, not project policy.
- **Conventional Commits**, enforced by `commitlint` (see `.commitlintrc.json`). The `subject-case` rule is relaxed so Dependabot's `Bump X from Y to Z` subjects pass.
- **Justfile is the canonical task runner.** `just` lists every recipe. Don't reach for `go test ./...` directly when `just test` exists — the recipe sets the race flag, the cover profile, and the right toolchain.
- **No manual pre-flight; lefthook + CI own it.** Don't run `just test`, `just lint`, `go test`, `golangci-lint run`, `go build`, or `markdownlint-cli2` manually before pushing. The repo's `lefthook.yml` is layered:
  - `pre-commit` — sub-second formatters on staged files (`gofumpt` / `goimports` / `markdownlint-cli2 --fix`).
  - `commit-msg` — Conventional-Commits via `commitlint`.
  - `pre-push` — once-per-push gate that mirrors the CI `check` + `lint` + `forbid-skip` jobs: `golangci-lint run ./...`, `markdownlint-cli2` verify, and the discipline greps (`t.Skip*`, "not implemented" in prod code, soft-assertions / silent recovers, `should_skip` overlays, escape-hatch primitives). Bypass with `LEFTHOOK=0 git push` for WIP branches.
  - CI runs the full test suite + the compat / e2e / mutation lanes the local hook intentionally doesn't.
  - New contributors run `just hooks-install` once after cloning; agents trust the hooks + CI and don't pre-flight manually.
- **Tests assert or are removed — never `t.Skip` / soft-assert / silent-recover / `should_skip` a test.** If a feature can't be exercised on the CI substrate (e.g. a CH function above the chDB floor), gate it at runtime and validate elsewhere (prod / e2e), never skip. The `forbid-skip` gate enforces this behaviourally; honest prose ("deferred", "skipped") describing correct version-gated code is fine — the removed `wording-tests` scan policed words, not behaviour.
- **Compatibility is the source of truth for all three heads.** The unified `compatibility.yml` workflow runs on pull_request + push-to-main + nightly + manual dispatch. All three jobs — `compatibility/prometheus`, `compatibility/loki`, `compatibility/tempo` — are required status checks on `main` (gate flip completed 2026-05-19; verified on 2026-05-21 with 9/9 consecutive green runs after the standalone `tempo-compatibility.yml` workflow was deleted and consolidated into this one). There are **no allow-lists**: the old `expected-failures.json` mechanism is deleted, and every diff against a reference backend is a real bug to fix at the source. The only pinned exclusion set is `compatibility/loki/upstream-skip-baseline.txt`, which records the corpus entries *upstream itself* marks `skip: true` (no reference baseline exists for them); the harness fails on any drift in that set (see `docs/compatibility.md`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsouza/cerberus](https://github.com/tsouza/cerberus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
