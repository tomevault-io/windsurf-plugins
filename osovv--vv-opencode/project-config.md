---
trigger: always_on
description: opencode, plugins, workflow, model-roles, hashline-edit, work-items, controller
---

# GRACE 4 Project Engineering Protocol

## Keywords
opencode, plugins, workflow, model-roles, hashline-edit, work-items, controller

## Annotation
Portable OpenCode workflow package with plugins and a Bun CLI for install, sync, semantic model-role assignment, controller-led workflow routing, safer editing, secrets redaction, and cross-device setup.

## GRACE 4 Source of Truth

This project uses the GRACE 4 `.grace` artifact model.

- Product and technical context: `.grace/context/*.xml`
- Current graph projection source: `.grace/graph/index.xml` plus routed graph documents such as `.grace/graph/main.xml`
- Current verification projection source: `.grace/verification/index.xml` plus routed verification documents such as `.grace/verification/main.xml`
- Active work: `.grace/changes/active/C-*/spec.xml` and `.grace/changes/active/C-*/plan.xml`
- Completed or terminal work: `.grace/changes/archive/C-*/*`

Historical Markdown notes under `docs/` are advisory references only. They are below explicit user instructions, source code, tests, `README.md`, and current `.grace` state.

## Project Snapshot

- package name: `@osovv/vv-opencode`
- CLI binary: `vvoc`
- primary runtime: `Bun`
- language: `TypeScript`
- CLI framework: `citty`
- public plugin exports:
  - `GuardianPlugin`
  - `HashlineEditPlugin`
  - `ModelRolesPlugin`
  - `SystemContextInjectionPlugin`
  - `WorkflowPlugin`
  - `SecretsRedactionPlugin`
- current command set includes: `init`, `install`, `sync`, `launch`, `status`, `doctor`, `version`, `role`, `preset`, `config validate`, `plugin list`, `plugin enable`, `plugin disable`, `patch-provider`, `completion`, `guardian config`, `upgrade`

## Repository Rules

1. `src/` is the source of truth for implementation behavior.
2. `dist/` is generated output. Never edit it manually.
3. Release and publishing policy is defined in `.grace/context/deployment.xml`; do not duplicate release-flow details here.
4. If CLI behavior, package exports, setup flow, or config locations change, update `README.md` in the same change.
5. This package manages user config, so prefer conservative, idempotent writes over aggressive rewrites.
6. If modules, public exports, data flows, verification strategy, commands, critical scenarios, or log markers change, update the relevant `.grace` graph and verification artifacts in the same change.

## Core Principles

### 1. Never Write Code Without a Contract
Before generating or editing any module, create or update its `MODULE_CONTRACT` with PURPOSE, SCOPE, INPUTS, and OUTPUTS. Code implements the contract, not the other way around.

### 2. Semantic Markup Is Load-Bearing Structure
Markers like `// START_BLOCK_<NAME>` and `// END_BLOCK_<NAME>` are navigation anchors, not decoration. They must be uniquely named, paired, and proportionally sized so one block fits inside an LLM working window.

### 3. GRACE Graph Is Always Current
`.grace/graph/index.xml` and routed graph documents are the project map. When you add a module, move a module, rename exports, or add dependencies, update `.grace/graph/*` so future agents can navigate deterministically.

### 4. Verification Is a First-Class Artifact
Testing, traces, and log anchors are designed before large execution waves. `.grace/verification/index.xml` and routed verification documents are the verification contract. Logs are evidence. Tests are executable contracts.

### 5. Top-Down Synthesis
Code generation follows:
`GraceRequirements -> GraceTechnology/Principles -> GraceGraph -> GraceVerification -> Code + Tests`

Never jump straight to code when requirements, architecture, or verification intent are still unclear.

### 6. Governed Autonomy
Agents have freedom in HOW to implement, but not in WHAT to build. Contracts, specs, plans, graph references, verification entries, and user instructions define the allowed space.

## Working Conventions

### CLI and packaging

- Local consumers of a project dependency should run `vvoc` through `bun x vvoc` or `bun run vvoc`.
- Root package exports must stay valid:
  - `@osovv/vv-opencode`
  - `@osovv/vv-opencode/plugins/guardian`
  - `@osovv/vv-opencode/plugins/hashline-edit`
  - `@osovv/vv-opencode/plugins/model-roles`
  - `@osovv/vv-opencode/plugins/system-context-injection`
  - `@osovv/vv-opencode/plugins/workflow`
  - `@osovv/vv-opencode/plugins/secrets-redaction`
- Local quality tooling uses `oxlint`, `oxfmt`, and `lefthook`.
- `lefthook` owns the `pre-commit` hook and should keep running lint + format checks.
- Before release, run:
  - `bun run typecheck`
  - `bun run lint`
  - `bun run fmt:check`
  - `bun test`
  - `bun run build`
  - `bun run pack:check`
  - `bun run release:check` when release/schema consistency matters

### Config safety

- OpenCode config remains in OpenCode-managed paths.
- vvoc-managed config must live in `$XDG_CONFIG_HOME/vvoc/` or project-local `.vvoc/`.
- `guardian.jsonc` may only be auto-rewritten when it is clearly managed by `vvoc`, unless the user explicitly forces overwrite.
- Persisted vvoc data must live in `$XDG_DATA_HOME/vvoc/`.
- Never silently clobber user-owned config.

### Documentation and GRACE sync

- If command names, flags, examples, install flow, or vvoc config paths change, update `README.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osovv/vv-opencode](https://github.com/osovv/vv-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
