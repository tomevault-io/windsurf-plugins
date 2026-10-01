---
trigger: always_on
description: > **Canonical statement.** This file is the sole always-mandatory human-readable
---


# AGENTS.md — The Gibson Operational Contract

> **Canonical statement.** This file is the sole always-mandatory human-readable
> authority for commit, PR, and merge behavior. Explanation and history under
> `docs/` are on-demand and non-normative. They must not add, drop, or weaken a
> rule stated here.

You are one agent in a fleet running the full SDLC on a target repository. This
file is identical for every runtime. Vendor adapters (`adapters/<vendor>/`) add
ergonomics; they never change these rules.

The policy-manifest candidate (`config/policy/candidates/gibson-core-v1.candidate.json`,
`authority=report-only`, `activated=false`) is a **checked mirror**, not authority,
until **#164**. `scripts/contract-authority.mjs` fails on drift and must not take
binding values from the candidate.

## Authority and mandatory load

**Fixed mandatory human-readable load (byte-budgeted):** this file only.

**Conditional session-start human-readable load:** when a role is dispatched,
also load that role's playbook (`playbooks/<role>.md`, replaced by
`local/playbooks/<role>.md` when present). If no role is named, the resolved
role is `builder` and `playbooks/builder.md` is that load (including default
assignment). When a non-role job is dispatched, also load that job's dispatch
prompt. That playbook is part of session-start load for the session — not
optional flavor text. Playbooks are role/job dispatch prompts and routing
mirrors; they must not add, drop, or weaken rules in this file. Fixed load
and conditional dispatch-prompt load are measured separately (see
`config/policy/mandatory-read-chain.v1.json`).

**Also at session start (not the fixed byte budget):**

1. `local/AGENTS.local.md` if present — fork overlay; **local wins**, except the
   Ask Contract may not be weakened, and human-gate entries may be extended
   locally but removed only by the fork's human owner, recorded in that fork's
   `memory/DECISIONS.md`.
2. The **target repo's** `AGENTS.md`.
3. Relevant entries in `memory/LESSONS.md` (tag-filter to role and area). Do not
   ingest the full ledger unless the task needs it. Attest the consult (Law 1
   sensor: `scripts/contract-read-check.mjs`).
4. Recall fleet memory (Mission Control `recall`, or `memory/` grep) before
   starting — someone may have solved or claimed this already.

Everything under `docs/`, helper templates, `paper/`, `HOW-IT-WORKS.md`,
adoption guides, findings, spikes, retrospectives, and historical narrative is
**on-demand and non-normative**. A linked explanation is not a second contract.

Machine-readable sources (activated or operational sensors — not the report-only
candidate):

| Topic | Authoritative / operational source |
|---|---|
| This contract + closed G1–G16 / roles / tiers / stages / pairs (prose) | `AGENTS.md` |
| Report-only enumeration mirror (not authority; until #164) | `config/policy/candidates/gibson-core-v1.candidate.json` |
| Review round caps | `config/review-round-caps.json` |
| Test-integrity (count ratchet, waivers) | `scripts/test-integrity.mjs` |
| Claim overlap / admission | `scripts/scope-overlap.mjs` |
| Green-gate runner | `scripts/gate.sh` |
| Local green-gate commands (machine twin) | `.agents/gate.json` |
| DCO local hook | `scripts/setup-hooks.sh` |
| Exact-head claim release | `scripts/release-claim.sh` |
| Fixed + conditional read-chain budget | `config/policy/mandatory-read-chain.v1.json` |
| Rule → home migration audit | `config/policy/rule-migration-audit.v1.json` |
| Per-role / per-job outputs / gates / prohibitions | `config/policy/role-contracts.v1.json` |

## Mission

Move well-scoped work through build, test, review, UX, security, merge, and
deployment **without stopping** except at this file's human gates. Ship small
units and leave the harness better.

## The Ten Laws

1. **Read before you act.** Load this file, then the session-start items above
   (including the dispatched role playbook when a role is named, or the job
   dispatch prompt when a job is named), before writing anything.
2. **Claim before you touch.** One issue = one claim = one worktree = one
   branch. Use `scripts/claim.sh` (adds the `agent-claimed` label and a PR-body
   claim). Check live claims first. Overlap → stop and coordinate, never race.
   A second lane on the same issue requires `--slice` and disjoint scope.
3. **Never edit in the canonical checkout.** All mutation happens in your own
   git worktree. The shared checkout is read-only, always.
4. **The green gate is absolute.** Before every commit: generate → typecheck →
   lint → test → build, with **zero new failures vs. your branch point**. Record
   the baseline when you branch (`scripts/gate-baseline.sh`). Pre-existing
   failures are not yours to inherit or to hide behind. Do not delete or skip
   tests to go green — `scripts/test-integrity.mjs` is canonical for that
   ratchet. **Exception — pure memory commits:** if the commit touches **only**
   files under `memory/` (and no product, script, CI, or playbook code), skip
   the green gate and CI loop. Still never put secrets in memory files. A commit
   that mixes memory with other paths follows Law 4 in full.
5. **Never grade your own homework.** You may not review, approve, or evaluate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-AIE/the-gibson](https://github.com/The-AIE/the-gibson) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
