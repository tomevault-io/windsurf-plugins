---
trigger: always_on
description: One engine room; PBC above; GT-KB Logbook below; crystals are how narration
---

# Repository Guidelines

## Where This Repo Sits (read before reasoning about boundaries)

One engine room; PBC above; GT-KB Logbook below; crystals are how narration
arrives; Farm is whose work arrives; LabWired is where hardware evidence comes
from. This CLI is the engine room — the enforcement middle. The **upper boundary**
is PBC (Product Behavior Contracts: what work is allowed). The **lower boundary** is the
GT-KB-derived evidence layer (the Logbook: "do not trust the narration — verify
against the evidence"). `agent-crystallize` crystals and similar session
artifacts are **narration formats arriving at** the lower boundary — untrusted
input in a parseable envelope, never a boundary and never trusted status.
Canonical taxonomy: `project-phoenix/docs/BULKHEAD_TAU_BOUNDARIES.md`. If a spec
or entry in this repo appears to contradict that doc, the doc wins — flag the
drift instead of propagating it.

Cross-repo vocabulary: **BN** = `~/Python/project-phoenix/BOTTLENECKS.md` — the
open-work board that schedules work across BT (Bulkhead Tau, which lives inside
`~/Python/project-phoenix/`) and this repo. BN's header carries the canonical
glossary; BN ≠ BT. Harnesses are **peers** (Claude Code, Codex, Antigravity, Grok,
local lanes — not ranked brands). Frontier seats are expected to **build into the
system**: recover this boundary map from the funnel, calibrate from Phoenix
failure-mode catalogs when working across repos, and leave residue (commits,
catalog entries, handoffs) under the human’s rules — see
`project-phoenix/docs/AGENT_AUDIT_PROTOCOL.md` § “Frontier seat fitness”.

## Concurrent Sessions & Ledger Identity

**Ruled 2026-08-11** (`session-coordination-protocol` task, Q2): when multiple sessions
write to this ledger at once, `--by`/`--assign`/`--review` use the **session-derived id**
(short form of the real Claude Code session id, e.g. `claude-019KSo7K` for
`session_019KSo7KhEUrNJGa1kSVeP8i`), not a role label. Role labels
(`claude-supervisor`, `claude-consultant`, `claude-builder`) were tried first and
drifted twice on this exact ledger — once when two sessions were assumed to be one
(`session-coordination-protocol` handoff-0001), once when a single session's own label
changed three times in 90 minutes. Session ids don't drift by construction; they're
also what the git commit trailers (`Claude-Session: https://claude.ai/code/session_...`)
already carry, which is how the original mixup got resolved in the end.

**Known gap in this ruling, found while implementing it, not resolved by it:** a
session-derived id is not guaranteed stable across a resume. `claude-consultant`'s role
was held by `session_0133KSgM` through 2026-08-09, then by `session_01Hzi1zP` from
2026-08-11 (`3b86ecb`) — same role, same continuity of work, different id. Treat the
table below as tracking **role continuity**, not asserting that an id is permanent.
When a session resumes under a new id and picks up a prior session's thread, add a row
rather than overwrite one, and say so in the handoff that continues the work.

**Recently active sessions** (populate/update as sessions come and go; this is not a
full historical audit — plenty of one-off sessions touched this repo before
2026-08-09 and aren't tracked here):

| Session id | Root / working dir | Recent work | Last active |
|---|---|---|---|
| `claude-01QBpGoE` | `~/operator-control-plane` | Front A infra, `session-coordination-protocol` original proposal (as `claude-supervisor`) | 2026-08-09 |
| `claude-0133KSgM` | `~/operator-control-plane` | Confound pilot passes 1-2, coordination-protocol counter-proposal (as `claude-consultant`) | 2026-08-09 |
| `claude-01Hzi1zP` | `~/operator-control-plane` | `front-e0-desktop-pack-review`, `LOCAL_LANE_CONTRACT_SPEC.md` power-cap fix (continuing `claude-consultant`'s role) | 2026-08-11 |
| `claude-01Q3rn3n` | `~/operator-control-plane` | Front G, pa-evidence Gate 1 adapter | 2026-08-10 |
| `claude-019KSo7K` | `~/Alignerr` (cross-repo, via SSH to desktop) | Front D dashboard verification, Front E0 desktop pack, Q1/Q4 ruling implementation | 2026-08-11 |
| `claude-01FSgUqu` | `~/.dotfiles` on **desktop** (cross-repo: this repo + `~/Python`) | Runner trace retention (`--trace-dir`), E1 27-cell desktop matrix + `FINDING.md`, gemma4:31b throughput correction across GOLD_STANDARD/Phoenix docs, SEAT-COST-001 cross-check | 2026-08-12 |

Registering a session-derived id as a harness (`.operator/harnesses/claude-<id>.yaml`)
is required only if something needs to `--assign`/`--review` to it — `--by` needs no
registration (Q1 ruling, same task). `.operator/` is gitignored and per-machine, so a
harness registered on z13 does not exist on desktop until copied there.

## Project Structure & Module Organization

This repository is a compact Python CLI project (requires Python ≥ 3.12).

| Path | Role |
|------|------|
| `operator` | Main ledger CLI (~6000 lines; most repo-local changes land here) |
| `opr` | Confirmation-gated governed REPL for local-model sessions |
| `operator-broker` / `authority_broker.py` | Standalone P3a authority broker |
| `operator-admin` / `authority_admin.py` | Root-managed P3b policy install/lifecycle |
| `authority_client.py` / `authority_projection.py` | Enrolled CLI ↔ broker integration |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blue-az/operator-control-plane](https://github.com/blue-az/operator-control-plane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
