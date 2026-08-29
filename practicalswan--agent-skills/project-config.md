---
trigger: always_on
description: This repository contains shared skills for GitHub Copilot, Claude Code, and
---

# AGENTS.md

This repository contains shared skills for GitHub Copilot, Claude Code, and
Codex, and this file defines the workspace-specific rules that should
apply to any AI agent operating in `C:\Users\LOQ\.copilot\skills`.

## Required Session Start Rule

- Every new session in this workspace must begin by reading `LESSON.md`.
- Treat `LESSON.md` as required startup context before analysis, planning,
  edits, validation, reviews, or advisory work.
- If `LESSON.md` is missing or unreadable, stop and report that blocker before
  continuing.

## Required Completion, Sync, And Publish Rule

- For every user-requested mutation task in this workspace, complete the
  requested work in `C:\Users\LOQ\.copilot\skills` first.
- After the work is complete, run the repo validation, then sync outward to the
  downstream skill folders every time.
- If the AI agent judges the result satisfactory, commit and push to GitHub
  without asking for another confirmation.
- Treat work as satisfactory only when validation passes, sync completes,
  the task is complete, no requested step was skipped, no required command was
  rejected, no unresolved secret/security/privacy issue remains, and the final
  diff matches the user's request.
- Elevate to the user before commit or push when there are security concerns,
  incomplete work, skipped steps, rejected or blocked required commands,
  validation/sync failures, unexpected unrelated dirty files that make
  staging unsafe, or any other reason the work is not satisfactory.
- For read-only or advisory tasks with no file changes, do not create empty
  sync, commit, or push churn; report that no mutation workflow was needed.

## Workspace Role

- Main branch: `C:\Users\LOQ\.copilot\skills`
- New maintained skills must be added or imported here first.
- Maintained skills live here and are synced outward to downstream targets.
- Copied official superpowers are tracked here for discovery and cross-client
  sync, but they are not maintained the same way as the catalog's maintained
  skills.

## Codex-Only Blender Skills Overlay

- The `arjun988/blender-skills` pack is an explicit exception to normal child promotion.
- Its 94 upstream skills plus the separately protected local
  `raw-scan-to-aaa-preserve-texture` entry (95 protected names total) must
  remain installed only under `C:\Users\LOQ\.codex\skills`, with its source
  checkout under `C:\Users\LOQ\.codex\vendor\blender-skills`.
- Never promote these skill names into this parent catalog and never sync them to `C:\Users\LOQ\.agents\skills` or `C:\Users\LOQ\.claude\skills`.
- `scripts/skill-registry.json` records the protected names and the Codex-only source configuration; generic promotion and sync tooling must honor that boundary.
- During parent source-maintenance or "update all skills" work, run `scripts/update-codex-local-blender-skills.ps1`. It fetches upstream, refreshes only the owned Codex copies and shared Blender references, updates the ownership manifest and source commit, and verifies that no Blender skill escaped to a forbidden root.

## Current Counts

Snapshot date: `2026-08-24`. Local overlay totals can differ by machine.

- Git-tracked catalog in this repository:
  - `237` tracked skill folders
  - `205` tracked maintained skills
  - `32` tracked copied official Superpowers
- Live local workspace snapshot, including local-only overlays such as
  `gws-*` and `recipe-*` when present:
  - `295` local skill folders detected
  - `263` local maintained skills detected
  - `32` local copied official Superpowers detected

Copied official superpowers are identified by the explicit
`copied_official_superpowers` list in `scripts/skill-registry.json`, not by
whether a skill folder has a `CHANGELOG.md`.

All `237` tracked skills use catalog `version: "2.0"`. The `166` pre-existing
tracked skills retain their prior catalog baselines; the 66 platform skills
retain their import provenance, and five Codex Router skills were promoted
from the personal Codex root. The catalog-wide maintenance baseline is
`last_updated: 2026-08-24`. The `58` local-only Google
Workspace overlays keep their upstream `version: "0.22.5"`
while receiving the same retained-client sections and maintenance date.

The tracked imports `docx`, `jupyter-notebook`, `pptx`, and `xlsx` now have
finalized canonical provenance. Child-path promotion is handled by
`scripts/promote-child-skills.py`, while `scripts/update-skill-registry.py`
refreshes provenance and the reference-source report.

The Tavily suite is sourced from `tavily-ai/skills` at commit
`ea5e8201b0d3ed9c10b70b71187589bd761fe2d2`. Keep its eight skills
self-contained, retain the local secret and prompt-injection safeguards, and
do not reintroduce removed-client integrations from upstream references.

The selected Matt Pocock skills are sourced from `mattpocock/skills` at the
current audited commit `6654f6b60cd9d5be8b54c6fafe44346dabeb3b76`. The import
keeps only eight
cross-client workflows that fill gaps for the user's OCR, storefront,
Three.js, and multi-repository work. Existing TDD, debugging, review,
implementation, planning, and skill-authoring equivalents remain canonical.

The 2026-08-14 source refresh audited current upstream heads and updated the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PracticalSwan/agent-skills](https://github.com/PracticalSwan/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
