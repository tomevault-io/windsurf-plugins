---
trigger: always_on
description: skill-bill governs authoring, routing, validation, installation, and measurement of agent skills. It ships shared orchestration, tooling, telemetry, workflow state, and shells for review, quality checks, feature work, verification, and PRs.
---

# AGENTS.md

## Project Context

skill-bill governs authoring, routing, validation, installation, and measurement of agent skills. It ships shared orchestration, tooling, telemetry, workflow state, and shells for review, quality checks, feature work, verification, and PRs.

Non-negotiable contracts:

- Authored source is `content.md` (plus governed sidecars); generated `SKILL.md` is install output. Listed-skill staging: `SKILL.md`, `.content-hash`, pointers, optional `native-agents/` — no `content.md` copy.
- Skill sources: `skills/<skill>/content.md`, optional `native-agents/`, contract-authorized sidecars.
- Platform behavior lives in manifest-declared packs under `platform-packs/<slug>/`.
- `orchestration/` owns shared routing, review, delegation, telemetry, workflow, and shell contracts.
- `agent-addons/<slug>/` contains only user-owned `agent-addon.yaml` and `content.md`.
- Generated support pointers, provider-specific native-agent outputs, and install staging artifacts are not committed.
- Discovery, install, routing, and validation stay dynamic and manifest-driven.
- Missing manifests, wrong contract versions, missing content, and missing required sections fail loudly with typed errors.
- Every fallback, degradation, or swallowed failure emits a record; see `docs/observability-policy.md`.

## Internal Skills

`internal-for: <parent>` in `content.md` frontmatter installs as `<skill-name>.md` sidecar in the parent directory (not listed); parent reads the sibling in-session. The only parent is `skill-bill`; pack specialists install as its unlisted sidecars and native-agent inputs, never as slash commands. Contract: `docs/skill-source-generation.md`.

## Active Fix Branch

Until `base/SKILL-380-phase-slot-strategies` is merged, make all fixes on that
branch. Use its worktree when a feature branch has implementation work in progress.

## Product Intent

`/skill-bill` is the only listed skill. Read [Runtime command guidance](docs/runtime-command-guidance.md) before invoking, planning, designing, changing, or reviewing runtime commands and workflows. It owns phase and operation behavior, retired-name compatibility, agent execution, and goal commit/checkpoint rules.

## Taxonomy

- `skills/` — canonical user-facing skill source; `skills/skill-bill/` is the only listed skill
- `platform-packs/<platform>/` — pack roots for code review and pack `validation_gate` quality-check argv; `addons/` flat pack-owned add-ons. Packs are excluded from goal-planning discovery; eligible `agent/history.md` and `agent/decisions.md` reach planning as a heading catalog only — bodies arrive for headings preplanning selected.
- `orchestration/contracts/` — runtime contract schemas

Naming (pack skills and sidecars): `bill-<capability>`; overrides `bill-<platform>-<base-capability>`; review areas `bill-<platform>-code-review-<area>`. Approved areas: `architecture`, `performance`, `platform-correctness`, `security`, `testing`, `api-contracts`, `persistence`, `reliability`, `ui`, `ux-accessibility`.

## Source And Generated Files

Read `docs/skill-source-generation.md` before changing skills, scaffolding, rendering, install staging, native-agent generation, or support pointers.

Forbidden in source: governed `SKILL.md` wrappers; generated support pointers (`shell-ceremony.md`, `telemetry-contract.md`, `stack-routing.md`, review/delegation/add-on pointers); provider-specific `*-agents/` outputs (`claude-agents/`, `codex-agents/`, `junie-agents/`, `cursor-agents/`).

Native-agent source is provider-neutral under `native-agents/agents.yaml` or `native-agents/<name>.md` with `contract_version` on new/rendered sources (older sources still parse for fixture migration). Extra authored guidance goes in `content.md` H2 sections — not sibling org files like `patterns.md` under `skills/<skill>/`.

Run `./install.sh` after changing source skills, renderer behavior, or support pointer generation.

## Platform Packs

Packs are the extension surface; routing and install read manifests, not hard-coded platform lists. Canonical shape: `orchestration/contracts/platform-pack-schema.yaml`. Schema changes land there first; `ShellContentLoader.buildPack` rejects malformed manifests via `InvalidManifestSchemaError`. Cross-field rules JSON Schema cannot express live in Kotlin under `x-coherence-checks`.

Per-repo customization: top-level custom fields allowed; runtime-consumed fields use `x-runtime-anchored: true`; non-anchored fields flow to `PlatformManifest.customFields`. Product vs extension: horizontal `skills/skill-bill/` and `.bill-shared` are protected; `platform-packs/<slug>/` (including shipped `kotlin`/`kmp`) are removable — no paired `skills/<platform>/` trees.

`kmp` covers Android and Kotlin Multiplatform on the Kotlin baseline. Its `validation_gate` owns quality-check commands, with no Kotlin fallback. `operation:verify` remains pre-shell.

## Runtime Contract Schemas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sermilion/SkillBill](https://github.com/Sermilion/SkillBill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
