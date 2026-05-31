---
trigger: always_on
description: Operational notes for working in this repo as an autonomous agent (Claude
---

# AGENTS.md

Operational notes for working in this repo as an autonomous agent (Claude
Code or otherwise). Designed as a load-bearing playbook: read it before
your first commit, link it from PRs that follow non-obvious workflows.

## Validation tier model

Three tiers, additive (`tier3 ⊃ tier2 ⊃ tier1`). Wall-clock + exact bug
coverage are documented in [`scripts/validate-timings.md`](scripts/validate-timings.md).
Short version:

| Tier  | When to run                                | Wall clock      | Run command                                       |
|-------|--------------------------------------------|------------------|---------------------------------------------------|
| tier1 | After every code change                    | ~10s             | `make validate TIER=tier1`                        |
| tier2 | Before commit / before PR                  | ~6m              | `make validate TIER=tier2 PAHO=/tmp/paho-testing` |
| tier3 | Before tag / weekly nightly                | ~12–18m          | `make validate TIER=tier3 PAHO=/tmp/paho-testing` |

What each tier closes that the prior tier misses:

- **tier1** — compile / vet / race-light unit tests / chart lint+template.
  Catches static bugs, helm rendering issues (typoed values keys, scientific-
  notation in int fields), small-package data races.
- **tier2** — full unit suite against a testcontainer Postgres + paho v3+v5
  conformance against a single broker. Catches SQL path regressions and
  MQTT protocol shape bugs (reason codes, retain, will-delay, takeover
  ordering).
- **tier3** — multi-broker paho via kind + 60s soak smoke. Catches cross-
  pod fanout bugs, deploy-shape regressions (helm-rendered manifests
  applied to a real apiserver), sustained-load issues.

In CI:

- `ci.yml` — covers tier1's surface on every PR (vet, govulncheck, test,
  test-race, helm lint, kind-based smoke). Already green is your gate to
  merge.
- `conformance-nightly.yml` — runs tier2 daily at 03:00 UTC. Doesn't gate
  PRs; informational regression detector.
- `soak-weekly.yml` — runs an in-cluster soak (and tier3 paho multi-broker,
  best-effort) Mondays 04:00 UTC. Fails on any QoS≥1 loss/dups.

## Worktree-based parallel agent workflow

This codebase has been built with parallel agents on a worktree-per-task
model. The pattern:

1. Each task gets its own git worktree under `.claude/worktrees/agent-<short-id>/`,
   on a branch named `worktree-agent-<short-id>`. This means N agents can
   `cd` into N different working copies and edit / build / test in parallel
   without stepping on each other's files.
2. Each worktree commits independently. The parent agent merges back to
   `main` via `git merge --no-ff worktree-agent-<id>` (see commits prefixed
   `merge worktree-agent-…` in the history).
3. Boundaries between worktrees are enforced by *task scope*, not by file:
   a worktree may touch any file relevant to its task, but the parent
   agent is responsible for sequencing merges so two worktrees aren't
   both modifying overlapping hot files concurrently.

Why this pattern works for this codebase:

- pgmqtt has a small set of dense files (engine, conn, janitor) and a long
  tail of small files (helm templates, tests, docs). Mechanical work on
  the tail (helm schema additions, doc fixes, lint passes) parallelises
  cleanly. Dense files are typically owned by one agent at a time.
- The merge cost is low: most worktrees touch ≤3 files outside their core
  task. Conflicts when they do happen are usually in `internal/engine/conn.go`,
  which we treat as a critical-section file: only one in-flight worktree
  may modify it.
- Each worktree's commits stay coherent — the merge commit on main names
  what each agent landed and why.

When NOT to use it:

- If the task is "make these three changes that all touch `engine.go`":
  that's a single-agent serial task. Don't spawn three worktrees that all
  fight for the same hot file.
- If the task crosses an architectural seam (e.g. "rip out the leader and
  go leaderless"): one agent, one branch. Coordinated rewrites lose
  coherence under parallelism.

## Common pitfalls observed

These have all bitten this codebase at least once. Listed here so future
agents can pattern-match instead of re-discovering.

### Helm scientific-notation rendering of int fields

Helm renders very large ints (≥10^7) in scientific notation when the
template doesn't force an explicit string conversion. This silently
truncated `maxPacketSize: 16777216` → `1.6777216e+07` → broker parsed
as `1` due to an Atoi tolerant cast that accepted the `1` prefix and
dropped the rest.

Fix: strict `strconv.ParseUint` in `internal/config/config.go` + helm
templates that pass numeric values through `int` (not `printf`'s default).
Caught by tier1 helm template + a parsing test; both are now part of
the standard validation run.

### kubectl port-forward instability under sustained load

`kubectl port-forward` uses an in-process userland forwarder that stalls
under backpressure. Symptoms in this repo:
- Paho conformance tests against a multi-broker kind cluster flake mid-run
  with `ConnectionRefused` on every fourth or fifth re-connect.
- `cmd/soak` reports loss / dups for QoS≥1 even when the broker is healthy,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bo0tzz/pgmqtt](https://github.com/bo0tzz/pgmqtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
