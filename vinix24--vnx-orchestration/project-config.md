---
trigger: always_on
description: <!-- VNX:BEGIN BOOTSTRAP -->
---

<!-- VNX:BEGIN BOOTSTRAP -->
## VNX Governance

This repository is governed by **VNX Glass Box Governance**: multi-agent orchestration with a human gate at every step and an append-only NDJSON receipt per dispatch.

The mechanism is not duplicated here. How the fabric works — the single-entry dispatch door and its lanes, review gates, the horizon planning layer, state resolution, and the report contract — lives in one canonical place so it can never drift out of a project file:

- **How the fabric works:** the canonical orchestrator role, `.claude/terminals/T0/role-orchestrator.md`, kept in sync fleet-wide by `vnx role sync`.
- **Runbooks + gotchas:** the `fabric-reference` skill.
- **Dispatch mechanics (lanes, provider routing, failure modes):** `docs/core/DISPATCH_RULES.md`.

Everything above this block describes *this project*. Everything the fabric does lives in the canonical role — never copy fabric mechanism back into this file, or the copy drifts the moment the fabric changes.
<!-- VNX:END BOOTSTRAP -->

<!-- The sections below live here because this repo IS the fabric. In a consumer
     project they do NOT belong in CLAUDE.md: the report contract reaches workers
     operationally via build_directive() at dispatch time, and lane detail lives in
     the canonical role + docs/core/DISPATCH_RULES.md. They stay outside the bootstrap
     block so `vnx role sync` / re-init never propagates them into consumer files. -->

## Mandatory Report Contract

**Every agent and worker MUST write a unified report on completing any task.**

This is how work enters the governed audit trail:
```
report on disk → receipt processor → t0_receipts.ndjson
```

Without a report, your work has no receipt and is invisible to governance.

Write to: `$VNX_DATA_DIR/unified_reports/<dispatch-id>.md`

Your report MUST contain these exact headings (aliases accepted):

| Required | Accepted aliases |
|---|---|
| `## Summary` | — |
| `## Changes` | `## Files Modified`, `## Work Completed` |
| `## Verification` | `## Test Results`, `## Evidence`, `## Tests` |
| `## Open Items` | — |

`## Summary` must be at least 50 non-whitespace characters. `## Open Items` may contain "None" explicitly. Include your dispatch ID as a plain-text or bold field (e.g. `Dispatch-ID: 20260601-213416-myfeature`). Full contract: `scripts/lib/report_body_contract.py`.

## Dispatch lanes

Two lanes ship on main; T0 picks per task. Full decision rule, provider strings, concurrency, and failure modes live in **`docs/core/DISPATCH_RULES.md`** (tmux-spawn lane detail: `docs/operations/TMUX_SPAWN_LANE.md`).

- **`scripts/lib/tmux_interactive_dispatch.py`** (default) — leaseless ephemeral, isolated worktree per dispatch, drives an interactive `claude` worker on the subscription. Use for parallel/independent feature work.
- **`scripts/lib/subprocess_dispatch.py`** — terminal-pinned (Wave 5 smart-context, lease, triple-gate). Opt in per terminal with `VNX_ADAPTER_T{n}=subprocess`. Use for single-worker PRs that benefit from prior-round findings, or work expected to run >30 min. **No Anthropic SDK** — only `subprocess.Popen(["claude", ...])`.

**Provider→lane rule (hard).** `claude`/Opus/Sonnet panelists and workers route via the **tmux-spawn lane** (subscription, June-15 escape) — NEVER `provider_dispatch` (it refuses claude: claude is not a provider-lane provider) and NEVER headless `claude -p` (API credits post-cutover). `kimi`/`glm`(litellm:zai)/`deepseek` route via `provider_dispatch.py`. Everything dispatches through the **single-entry door** (`vnx dispatch`), which decides the lane; calling a lane script directly is a side door (PR-12 consolidates the remaining ones, incl. the plan-gate panel). The plan-first gate (`plan_gate_panel.py`) honors this split.

For full documentation: `docs/`

<important if="working on schemas/migrations">
ADR-007 binding: every new central-DB table requires composite UNIQUE/PK over project_id.
See `docs/governance/decisions/ADR-007-multitenant-project-id-stamping.md`.
T0 must cite this explicitly in review-gate prompts.
</important>

<important if="working on review-gates or codex/kimi/gemini providers">
Per CC-COMMUNITY-SYNTHESIS-2026-05-29.md: codex for strict diff-mode, kimi for synthesis/operational angle.
Parallel review pattern proven 3x. Raw vs gate-routed dispatch = different audit trail — audit concern applies.
</important>

<important if="working on dispatch infrastructure or subprocess adapter">
Wave 6 elastic pool shipped 2026-05-16 (ADR-018, 9 PRs). Use `bin/vnx pool {status,scale,config,reap}`.
Backward-compat: terminal-pin via subprocess_dispatch.py still works.
SubprocessAdapter path: `scripts/lib/subprocess_adapter.py` + `scripts/lib/subprocess_dispatch.py`.
Single dispatch entry is the door (`vnx dispatch`): decision-tree enforced in code + side-door blocking.
Dispatch mechanics, lanes, and failure modes: `docs/core/DISPATCH_RULES.md`.
No T0 role file loaded (autonomous/project-root T0)? The staging flow for a governed dispatch
(track → central `stage_spec_bundle` → dry-run → fire → post-merge `link-pr`) is
`docs/core/DISPATCH_RULES.md` §12 "Autonomous dispatch — the staging flow".
</important>

<important if="working on receipt processor or governance/audit trail">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vinix24/vnx-orchestration](https://github.com/Vinix24/vnx-orchestration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
