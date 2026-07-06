---
trigger: always_on
description: skill-bill is a governed platform for authoring, routing, validating, installing, and measuring AI-agent skills. It ships shared orchestration, validators, installers, scaffolding, telemetry, workflow state, and stable base shells for review, quality checks, feature work, feature verification, and PR descriptions.
---

# AGENTS.md

## Project Context

skill-bill is a governed platform for authoring, routing, validating, installing, and measuring AI-agent skills. It ships shared orchestration, validators, installers, scaffolding, telemetry, workflow state, and stable base shells for review, quality checks, feature work, feature verification, and PR descriptions.

Non-negotiable contracts:

- Authored governed skill source is `content.md`; generated `SKILL.md` wrappers are runtime/install output.
- Source skill directories under `skills/<skill>/` contain only `content.md` plus optional `native-agents/`.
- Platform behavior lives in manifest-declared platform packs under `platform-packs/<slug>/`.
- `orchestration/` is the shared source of truth for routing, review, delegation, telemetry, workflow, and shell contracts.
- Generated support pointer files, provider-specific native-agent outputs, and installed staging artifacts are not committed.
- Discovery, install, routing, validation, and desktop surfaces stay dynamic and manifest-driven.
- Missing manifests, wrong contract versions, missing content files, and missing required sections fail loudly with typed errors.

## Product Intent

`bill-feature-task` is the feature-task router: it accepts `mode:runtime` (default) or `mode:prose`, presents one confirmation gate, then delegates. `bill-feature-task-prose` runs the full phase loop in-session. `bill-feature-task-runtime` launches the foreground `skill-bill feature-task` driver with durable workflow state, telemetry, platform packs, add-ons, and native subagents.

Bundled skills and reference packs are defaults, not the framework boundary. Teams may delete, fork, or replace them while retaining governed source shape, generated-output boundaries, manifests, install staging, validators, dynamic discovery, and loud-fail behavior.

## Taxonomy

- `skills/`: canonical user-facing skill sources.
- `skills/<platform>/`: legacy/pre-shell platform overrides for families not yet moved to platform packs.
- `platform-packs/<platform>/addons/`: flat pack-owned add-ons applied after routing.
- `platform-packs/<platform>/`: user-owned pack roots for code review and quality-check behavior.
- `orchestration/contracts/`: runtime contract schemas.

Naming:

- Base skills: `bill-<capability>`
- Platform overrides: `bill-<platform>-<base-capability>`
- Platform review areas: `bill-<platform>-code-review-<area>`
- Approved areas: `architecture`, `performance`, `platform-correctness`, `security`, `testing`, `api-contracts`, `persistence`, `reliability`, `ui`, `ux-accessibility`

## Source And Generated Files

Read `docs/skill-source-generation.md` before changing skills, scaffolding, rendering, install staging, native-agent generation, or support pointer behavior.

Generated files forbidden in source:

- governed `SKILL.md` wrappers under `skills/` or platform-pack skill directories
- generated support pointers such as `shell-ceremony.md`, `telemetry-contract.md`, `stack-routing.md`, review/delegation pointers, and add-on pointers
- provider-specific `claude-agents/`, `codex-agents/`, `opencode-agents/`, and `junie-agents/` outputs

Native-agent source is provider-neutral and lives under `native-agents/agents.yaml` or `native-agents/<name>.md`. New and rendered sources include `contract_version`; the parser still accepts older sources for gradual fixture migration.

If a skill needs more authored guidance, add H2 sections to `content.md`. Do not add extra organization files such as `patterns.md`, `reference.md`, or `audit-rubrics.md` under `skills/<skill>/`.

Run `./install.sh` after changing source skills, renderer behavior, or support pointer generation so local agent installs use the new staging hash.

## Platform Packs

Platform packs are the extension surface. Any maintained pack in this repo is valid when it follows the governed contract. Routing and install flows read pack manifests rather than hard-coded platform lists.

The canonical shape for `platform-packs/<slug>/platform.yaml` is `orchestration/contracts/platform-pack-schema.yaml` (Draft 2020-12 YAML-authored JSON Schema). Field additions, type changes, constraints, and enums land there first. `ShellContentLoader.buildPack` rejects malformed manifests through `InvalidManifestSchemaError`.

The shell contract version is `1.1`. `SHELL_CONTRACT_VERSION` and the schema `contract_version.const` are pinned by `PlatformPackSchemaContractVersionTest`.

Cross-field rules JSON Schema cannot express live in Kotlin and are documented under `x-coherence-checks`, including slug parity, declared-area parity, pointer uniqueness, baseline composition, and governed add-on usage.

Per-repo customization:

- top-level custom fields are allowed in `platform-pack-schema.yaml`
- runtime-consumed top-level fields carry `x-runtime-anchored: true`
- `PlatformPackSchemaAnchoredBijectionTest` enforces schema-to-Kotlin parity
- non-anchored top-level fields flow into `PlatformManifest.customFields`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oila-gmbh/skill-bill](https://github.com/oila-gmbh/skill-bill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
