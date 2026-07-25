---
trigger: always_on
description: Delete buttons delete immediately. No `confirm()`, no `window.confirm()`, no modal `showWarningMessage`, no two-click patterns, no "Are you sure?". The user has demanded this repeatedly. Buttons are deliberately hard to misclick.
---

# Switchboard — agent rules

## NEVER add confirmation dialogs. NO EXCEPTIONS.

Delete buttons delete immediately. No `confirm()`, no `window.confirm()`, no modal `showWarningMessage`, no two-click patterns, no "Are you sure?". The user has demanded this repeatedly. Buttons are deliberately hard to misclick.

Also a hard technical reason: `window.confirm()` is a **silent no-op in VS Code webviews** (sandboxed iframe without `allow-modals` — it always returns `false`). Any confirm gate added to `src/webview/planning.js`, `src/webview/kanban.html`, etc. makes the button do *literally nothing*. This exact bug broke the kanban delete-plan button (fixed 2026-06-11).

If you find a confirm gate in this codebase, it is a bug — remove it. Multi-choice decision dialogs (e.g. 3-way conflict resolution) are allowed; plain confirm gates are not.

## Build

- `npm run compile` (webpack) builds to `dist/`, but **`dist/` is NOT used during development or testing**. All testing is done via an installed VSIX — nothing is served from the repo's `dist/` directory. Do NOT audit, check, or flag `dist/` staleness during reviews or verification. Treat `src/` as the source of truth. `npm run compile` is only needed when producing a VSIX for release.

## Users & migrations

- **Published extension, ~4,000 installs**, many on much older versions. The dividing line is whether the state **shipped in a released version**:
  - State/files/settings that exist in any released version MUST be migrated on change: import before deleting, archive legacy files as `*.migrated.bak` rather than unlinking, preserve unknown/legacy keys instead of dropping them, and never assume a prior migration "already ran" for the install base.
  - Features that have only ever existed in unreleased dev work can take clean breaks — no migrations, no compat shims.
- When unsure whether something shipped, assume it did and migrate — a no-op migration costs nothing; a missing one destroys user data.

<!-- switchboard:claude-protocol:start -->
# CLAUDE.md - Switchboard Protocol

> **Claude Code note.** The Switchboard protocol below was authored for the Antigravity host. In Claude Code:
> - `view_file <path>` → use the **Read** tool.
> - `send_message` and role-routing (reviewer, lead, etc.) are **Antigravity-only** — ignore them here.
> - To run a workflow, invoke its native slash command (e.g. `/switchboard`, `/switchboard-cloud`, `/switchboard-remote`, `/switchboard-memo`) or read the skill at `.claude/skills/<name>/SKILL.md`.
> - The ClickUp / Linear / kanban skills shell out via `.agents/skills/_lib/sb_api_call.sh` and work as-is, provided the Switchboard extension (and its API server) is running.

---

# AGENTS.md - Switchboard Protocol

## 🚨 STRICT PROTOCOL ENFORCEMENT 🚨

This project relies on **Switchboard Workflows** defined in `.agents/workflows`.

**Rule #1**: If a user request matches a known workflow trigger, you **MUST** execute that workflow exactly as defined in the corresponding `.md` file. Do not "wing it" or use internal capability unless explicitly told to ignore the workflow.

**Rule #2**: You MUST NOT call `send_message` with unsupported actions. Only `submit_result` and `status_update` are valid (see Code-Level Enforcement below). The tool will reject unrecognized or unauthorized actions.

**Rule #3**: The `send_message` tool auto-routes actions to the correct recipient based on the active workflow. You do NOT need to specify a recipient. If the workflow requires a specific role (e.g. `reviewer`), ensure an agent with that role is registered.

### Workflow Registry

| Trigger Words | Workflow File | Description |
| :--- | :--- | :--- |
| `/switchboard` | **`switchboard.md`** | **The local management console** — drive the board, plans, features, dispatch, and automation while the VS Code extension is running. The primary front door; start here when unsure. |
| `/switchboard-cloud` | **`switchboard-cloud.md`** | Cloud-VM planning brake — plan first, do not auto-code in a remote VM. |
| `/switchboard-remote` | **`switchboard-remote.md`** | Remote Switchboard control — drive plans via Linear or Notion when the local machine is off. |
| `/switchboard-memo`, "start memo capture" | **`switchboard-memo.md`** | Memo capture mode — append-only, no analysis. Enter via `/switchboard-memo` or by saying "start memo capture". Exit with `process memo`. Edit entries with `edit N: <text>`. |

These four are the ONLY user-typeable workflow commands. Internal, extension-dispatched workflows are no longer slash commands: `improve-plan`, `improve-feature`, `accuracy`, and `switchboard-orchestrator` live as stripped skills under `.agents/skills/<name>/SKILL.md`, read by the extension by path (the orchestrator persona is system-launched from the AUTOMATION tab's Start orchestrator; never invoke it ad hoc).


### ⚠️ MANDATORY PRE-FLIGHT CHECK

Before EVERY response, you MUST:

1. **Scan** the user's message for explicit workflow commands from the table above (prefer `/workflow` forms).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TentacleOpera/switchboard](https://github.com/TentacleOpera/switchboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
