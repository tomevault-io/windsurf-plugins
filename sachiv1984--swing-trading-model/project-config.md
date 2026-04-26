---
trigger: always_on
description: Before doing anything else:
---

# CLAUDE.md — System Anchor

## 0. On Every Session Start

Before doing anything else:

1. Read `.claude_current_state.json`
2. Report the `active_cycle`, `status`, and `engine` to the user
3. If status is `Blocked`: do not proceed with any governed routine — surface the open escalations and halt
4. If context has been compacted (`/compact` was used), re-read `.claude_current_state.json` and the active prompt file before proceeding

Do not infer state from conversation history. Always read the file.

---

## 1. Governance Engines (Demand-Loaded)

These prompts are **not** always-on context. They are loaded by Claude Code when the corresponding command is issued. Each prompt file contains full instructions — read the file, then execute.

| Command | Prompt File | Phase / Trigger |
|---------|-------------|-----------------|
| `run ideas [--window-id <id>] [--mode strict\|standard]` | `claude/system/idea_intake_prompt.md` | Phase 0 — Idea intake (optional, before rebalance) |
| *(auto)* | `claude/system/idea_intake_prompt.md` | Invoked as STEP -1.6 of `run roadmap` when fewer than 20 open ideas (`Submitted` or `Parked-cycle-<n>`) exist in `claude/ideas/submissions/`. Standalone `run ideas` remains supported for explicit window control. |
| `run roadmap --item-id <id> --item-name <n> [--date YYYY-MM-DD] [--dry-run]` | `claude/system/roadmap_prompt.md` | Phase 1 — Roadmap rebalance (roadmap item completed) |
| `run roadmap --reason "scheduled" [--date YYYY-MM-DD] [--dry-run]` | `claude/system/roadmap_prompt.md` | Phase 1 — Roadmap rebalance (scheduled review, no completion event required) |
| `manage roadmap [--dry-run]` | `claude/system/roadmap_management_prompt.md` | Phase 1M — Retire completed items, flag stale |
| `groom backlog [--dry-run]` | `claude/system/backlog_management_prompt.md` | Phase 1M — Archive completed backlog items, health check |
| `plan release --version <vX.Y> [--date YYYY-MM-DD]` or `run planning v<version>` | `claude/system/release_planning_prompt.md` | Phase 1B — Release planning |
| `run design-gate --cycle <cycle_id> [--dry-run]` | `claude/system/design_gate_prompt.md` | Phase 1.5 — Design gate (after Phase 1B, before Phase 2) |
| `plan sprint [--cycle <cycle_id>] [--mode strict\|standard] [--dry-run]` | `claude/system/sprint_planning_prompt.md` | Phase 2 — Sprint planning |
| `amend cycle --cycle <cycle_id> --reason <reason> [--mode strict\|standard]` | `claude/system/amendment_cycle_prompt.md` | Amendment — Emergency only, before Phase 2 seals |
| `run sprint [--cycle <cycle_id>] [--epic <EPIC-xx>] [--item <ST-xx>] [--mode strict\|standard]` | `claude/system/execution_prompt.md` | Phase 3 — Sprint execution |
| `run delivery verification [--cycle <cycle_id>] [--mode strict\|standard]` | `claude/system/delivery_verification_prompt.md` | Phase 4 — Delivery verification |
| `run post-ship [--cycle <cycle_id>] [--mode strict\|standard] [--dry-run]` | `claude/system/post_ship_closure.md` | Post-Ship — Close cycle, apply lessons learnt |
| `sync gh` | *(inline — see §4)* | Sync backlog slice to GitHub Issues |
| `run audit` | `claude/audit.py` | Governance — lifecycle audit (every 3 cycles; output filed as `claude/cycles/<cycle_id>/audit_report_AUD-<date>.md` Class 3) |

When a command is issued, **read the prompt file first**, then begin execution. Do not rely on memory of the prompt's contents from a previous session.

---

## 2. Governance Non-Negotiables (Always Active)

These rules apply in every session regardless of which engine is running:

- **Never modify sealed artefacts.** Files marked `sealed: true` in state.json, or in `Published` state, are immutable.
- **Never modify governance files** unless explicitly instructed by the relevant prompt: `claude/charter/`, `claude/strategy/`, `claude/system/`. Exception: `claude/system/prompt_change_log.md` may be appended by the roadmap engine's STEP 11 when action-now patches are applied under Head of Specs Team sign-off.
- **Never merge a PR autonomously.** QA sign-off and Product Owner acceptance are always required.
- **Delivery pressure does not override governance.** No timeline instruction changes a hard gate.
- **Commit format is non-negotiable:** `[EPIC-xx][ST-xx] <description>` on all commits to `exec/**` branches. **When two stories are implemented in the same commit, all story IDs must appear in the message:** `[EPIC-xx][ST-xx][ST-yy] <description>`. Omitting a story ID prevents `governance_sync.yml` from closing that story's GitHub issue automatically.
- **PR title format is non-negotiable:** `[EPIC-xx] <description>` — required by `quality_gate.yml`.
- **Bash commands that write files outside an active prompt's write scope are prohibited**, even as side-effects.
- **Document section headings must be descriptive and human-readable.** Task IDs, story IDs, and ticket references (e.g. `TASK-11`, `ST-xx`) must never appear in headings. They belong only in metadata blocks, changelogs, or inline cross-references.
- **Every new API endpoint must be added to `docs/reference/openapi.yaml` in the same commit as the contract.** Any new `## METHOD /path` heading added to a file in `docs/specs/api_contracts/` must have a corresponding entry in `docs/reference/openapi.yaml`. Omitting this fails the OpenAPI Drift Detection gate and blocks all PRs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sachiv1984) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
