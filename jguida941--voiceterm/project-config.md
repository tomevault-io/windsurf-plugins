---
trigger: always_on
description: This file is the canonical SDLC, release, and AI execution policy for this repo.
---

# Agents

This file is the canonical SDLC, release, and AI execution policy for this repo.
If any process docs conflict, follow `AGENTS.md` first.

## Purpose

VoiceTerm is a polished, voice-first overlay for AI CLIs.
Primary support: **Codex** and **Claude Code**.
Gemini CLI remains experimental.

Goal of this file: give agents one repeatable process so every task follows the
same execution path with minimal ambiguity.

## Source-of-truth map

| Question | Canonical source |
|---|---|
| What are we executing now? | `dev/active/MASTER_PLAN.md` |
| What active docs exist and what role does each play? | `dev/active/INDEX.md` |
| Where is active-doc execution authority vs reference-only scope defined? | `dev/active/INDEX.md` (`Role`, `Execution authority`, `When agents read`) |
| Where is consolidated Theme Studio + overlay visual planning context? | `dev/active/theme_upgrade.md` |
| Where is long-range phase-2 research context? | `dev/deferred/phase2.md` (bridge at `dev/active/phase2.md`) |
| Where is the `devctl` reporting + CIHub integration roadmap? | `dev/active/devctl_reporting_upgrade.md` |
| Where is the autonomous loop + mobile control-plane execution spec? | `dev/active/autonomous_control_plane.md` |
| Where is the loop-output-to-chat coordination runbook? | `dev/active/loop_chat_bridge.md` |
| Where is the Rust workspace path/layout migration execution plan? | `dev/active/rust_workspace_layout_migration.md` |
| Where is the naming/API cohesion execution plan? | `dev/active/naming_api_cohesion.md` |
| Where is the IDE/provider adapter modularization execution plan? | `dev/active/ide_provider_modularization.md` |
| Where is the pre-release architecture/tooling cleanup execution plan? | `dev/active/pre_release_architecture_audit.md` |
| Where is the consolidated full-surface audit evidence used by that plan? | `dev/active/audit.md` (reference-only) |
| Where is the raw multi-agent audit merge transcript for that evidence set? | `dev/active/move.md` (reference-only supporting evidence) |
| Where are federated internal repo links/import rules (`code-link-ide`, `ci-cd-hub`)? | `dev/integrations/EXTERNAL_REPOS.md` |
| Where do we track repeated manual friction and automation debt? | `dev/audits/AUTOMATION_DEBT_REGISTER.md` |
| Where is the baseline full-surface audit runbook/checklist? | `dev/audits/2026-02-24-autonomy-baseline-audit.md` |
| Where are audit metrics definitions (AI vs script share, automation coverage, charts)? | `dev/audits/METRICS_SCHEMA.md` |
| How do we run parallel multi-agent worktrees this cycle? | `dev/active/MULTI_AGENT_WORKTREE_RUNBOOK.md` |
| Where are `devctl` command semantics and examples? | `dev/scripts/README.md` |
| Where is the devctl automation playbook? | `dev/guides/DEVCTL_AUTOGUIDE.md` |
| Where is MCP-to-devctl architecture alignment and extension policy? | `dev/guides/MCP_DEVCTL_ALIGNMENT.md` |
| Where is the remediation scaffold template used by guard-driven Rust audits? | `dev/config/templates/rust_audit_findings_template.md` |
| What user behavior is current? | `guides/USAGE.md`, `guides/CLI_FLAGS.md` |
| What flags are actually supported? | `rust/src/bin/voiceterm/config/cli.rs`, `rust/src/config/mod.rs` |
| How do we build/test/release? | `dev/guides/DEVELOPMENT.md`, `dev/scripts/README.md` |
| Where is the developer lifecycle quick guide? | `dev/guides/DEVELOPMENT.md` (`End-to-end lifecycle flow`, `What checks protect us`, `When to push where`) |
| Where are clean-code and Rust-reference rules defined? | `AGENTS.md` (`Engineering quality contract`), `dev/guides/DEVELOPMENT.md` (`Engineering quality review protocol`) |
| What process is mandatory? | `AGENTS.md` |
| What architecture/lifecycle is current? | `dev/guides/ARCHITECTURE.md` |
| Where are CI lane implementations and release publishers? | `.github/workflows/` |
| Where is the plain-language workflow guide? | `.github/workflows/README.md` |
| Where is process history tracked? | `dev/history/ENGINEERING_EVOLUTION.md` |

## Instruction scope and precedence

When multiple instruction sources exist, apply this precedence:

1. Session-level system/developer/user instructions.
2. The nearest `AGENTS.md` to the files being edited.
3. Ancestor `AGENTS.md` files (including repo root).
4. Linked owner docs from the source-of-truth map.

If subtrees require different workflows, add nested `AGENTS.md` files and keep
them scoped to that subtree.

## Autonomous execution route (required)

Use this route to run end-to-end without ambiguity:

1. Load `dev/active/INDEX.md`, then `dev/active/MASTER_PLAN.md`.
2. Use `INDEX.md` role/authority fields to decide which active docs are required:
   - `tracker` is execution authority.
   - `spec` is read when matching MP scope is in play.
   - `runbook` is read for active multi-agent cycles.
   - `reference` is context-only; do not treat as execution state.
3. Select task class in the router table and run the matching command bundle.
4. Apply risk-matrix add-ons for touched runtime risk classes.
5. Run docs-governance/self-review/end-of-session checklist before handoff.

## Mandatory 12-step SOP (always)

Run this sequence for every task. Do not skip steps.

1. Run session bootstrap checks and load `dev/active/INDEX.md` (`bundle.bootstrap`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jguida941/voiceterm](https://github.com/jguida941/voiceterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
