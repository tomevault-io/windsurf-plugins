---
trigger: always_on
description: > Shared source of truth for non-provider-specific repository rules.
---

# AGENTS.md - Rules for AI Coding Agents

> Shared source of truth for non-provider-specific repository rules.
> Codex prompts read this file only as runtime instructions; do not load `CLAUDE.md` or `GEMINI.md` for normal Codex startup.
> `CLAUDE.md` and `GEMINI.md` contain provider-specific context and may defer here for shared rules.

---

## Operator Contract (binding for ALL agents — read before acting)

The operator's working contract is `agents_extensions/shared/rules/operator-expectations.md`
(deploy copies: `.codex/rules/` · `.agent/rules/` · `.gemini/rules/`; served FIRST at
`GET http://localhost:8765/api/rules`). Its items are the tie-breakers when instructions
conflict. Digest — read the full file for the binding wording:
**1** Quality: no shortcuts, no threshold-lowering, no "for now" · **2** Best practices +
root-cause fixes · **3** Git/GitHub hygiene (**layout A**): primary non-bare on `main`
(human + services); agents only in `.worktrees/dispatch/<agent>/<task>/`; bare primary =
heal-as-bug; PRs + `X-Agent` · **4** Use the whole fleet; review gate = independent
**cross-family** reviewer (discussion ≠ review) · **5** Route by model × harness fit ·
**6** Limits: substitute lanes, NOTE substitutions · **7** Tool-backed claims only;
UA word/stress/morphology VESUM/`sources`-verified · **8** Clean code + current docs ·
**9** Max UA immersion EXCEPT A1 · **10** Drive within approved scope · **11** Repo hard
gates bind · **12** **Advisor/operator approval gate**: no architecture, layout, or
process decisions without present-tense **operator** or designated **advisor** approval
(current advisors: **Fable**, **Sol** — roster may change; check `/api/rules`).

### Fleet-comms mid-cutover (binding for standalone TUI/UI drivers; #5512 / #5632)

Full text: `agents_extensions/shared/rules/fleet-comms-coordination.md` (also in
`GET /api/rules`). Method: skill **`drive-epic`**; operator seats:
`docs/runbooks/epic-orchestrator-roster.md` + `./start-<model>-drive.sh <epic>`.
Digest: prefer `.venv/bin/python -m scripts.fleet_comms plane-status` +
`review-pr <PR_NUMBER>` / `publish-review-verdict` for topology and formal CF.
Plane modes are only `off|shadow|dual_write` — **file dual-write stays authoritative
in every mode** (`dual_write` is shadow/mirror, not authority cutover). Do not invent
a competing design or flip plane / retention apply / `formal_review_eligible` without
operator/advisor GO. Codex is a review/coding seat, **not** an epic-driver loop.
Stream leases are already claimed by launchers.

---

## Project Research Registry — Orchestrator Duty (binding)

Before every delegated task, the accountable orchestrator must deliberately classify
its functional role, task family, track, and owned paths. For scoped work, pass every
known dimension through `--research-role`, `--research-task-family`,
`--research-track`, and repeatable `--research-owned-path`; never infer them from the
provider or agent name. A genuinely generic or unknown task omits all research flags
and remains pointer-free. A surfaced pointer is not proof of consumption: research
claimed as used requires an attributed record fetch while the task is active. Registry
delivery remains fail-open, but forgetting this classification is not an acceptable
generic-task classification. Full contract and examples:
`agents_extensions/shared/rules/workflow.md` § Project Research Registry.

---

## Work Intake — Stream Epics (binding for ALL orchestrators; #4708)

Every open GH issue belongs to **exactly one stream epic** — registry:
`scripts/config/issue_streams.yaml` (streams → epic numbers). Your work queue is YOUR
stream's epic, not the global issue list. Codex UI track drivers: your curriculum-track
work hangs off its stream epic (seminars-folk #2836, seminars-bio #4431, core-quality
#4274, …) — pick up from there, and link every issue you create to its stream epic at
creation (native sub-issue preferred; `#N` checklist line in the epic body accepted).
Unlinked issues are flagged as ORPHANS at every agent's cold start (session-setup 11b)
and at `GET /api/issues/streams`. When a PR fixes an issue, close it with evidence —
and if scope remains, split it into a new linked issue BEFORE the auto-close keyword
eats it. Full protocol: `agents_extensions/shared/rules/workflow.md` § Work intake
(served at `/api/rules`).

## Global Codex Operating Rules

- Start every task at the repository root and run `git status --short --branch` before editing. **This is a read-only preflight/orientation step only.** Do not use this as permission to implement from the primary checkout.
- **The primary checkout is strictly read-only.** Do not drop scratch files, test scripts, or command outputs into its root directory under any circumstances. If you need a temporary file for discovery or testing, put it in your assigned worktree or an ignored scratch directory (like `batch_state/`).
- **Any implementation edit, branch work, commit, or PR MUST happen from a worktree (`.worktrees/dispatch/<agent>/<task>/`)** unless the user explicitly authorizes an exception. This prose rule is a reminder/backstop; mechanical enforcement is tracked in #4444-#4450.
- Preserve user and unrelated changes; do not revert, delete, or clean up work outside the task.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [learn-ukrainian/learn-ukrainian.github.io](https://github.com/learn-ukrainian/learn-ukrainian.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
