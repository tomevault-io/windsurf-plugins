---
trigger: always_on
description: This `AGENTS.md` applies to the entire repository.
---

# DeepScientist Repository Guide

This `AGENTS.md` applies to the entire repository.

It is the repository-level guide for coding agents and maintainers working in this checkout.
It should describe the code and docs that actually exist here today.

When code and docs diverge, prefer the current runtime behavior and tests, then update the docs in the same change.

## Mission

Build DeepScientist as a small, local-first research operating system that:

- runs on the user's machine by default
- installs cleanly through npm
- keeps the authoritative runtime in Python
- uses prompt-led and skill-led workflow control
- stores durable state in files plus Git
- keeps one quest as one Git repository
- supports the full research loop inside one quest workspace

The target is a focused core runtime, not a large platform.

## Read First

Do not start from memory. Start from the files that actually exist.

Recommended reading order for coding agents:

1. `README.md`
2. `docs/en/README.md`
3. `docs/en/90_ARCHITECTURE.md`
4. `docs/en/91_DEVELOPMENT.md`
5. the smallest subsystem-specific doc set that matches the task
6. the actual implementation under `src/`
7. the relevant tests under `tests/`

Important current docs by topic:

- product and docs index:
  - `README.md`
  - `docs/en/README.md`
  - `docs/zh/README.md`
- maintainer architecture and workflow:
  - `docs/en/90_ARCHITECTURE.md`
  - `docs/en/91_DEVELOPMENT.md`
- user-facing architecture and workflow:
  - `docs/en/13_CORE_ARCHITECTURE_GUIDE.md`
  - `docs/en/14_PROMPT_SKILLS_AND_MCP_GUIDE.md`
  - `docs/en/06_RUNTIME_AND_CANVAS.md`
  - `docs/en/07_MEMORY_AND_MCP.md`
- runtime and settings:
  - `docs/en/00_QUICK_START.md`
  - `docs/en/01_SETTINGS_REFERENCE.md`
  - `docs/en/05_TUI_GUIDE.md`
  - `docs/en/09_DOCTOR.md`
  - `docs/en/31_LOCAL_BROWSER_AUTH.md`
  - `docs/en/20_WORKSPACE_MODES_GUIDE.md`
- runner/provider setup:
  - `docs/en/15_CODEX_PROVIDER_SETUP.md`
  - `docs/en/24_CLAUDE_CODE_PROVIDER_SETUP.md`
  - `docs/en/25_OPENCODE_PROVIDER_SETUP.md`
  - `docs/en/27_KIMI_CODE_PROVIDER_SETUP.md`
  - `docs/en/21_LOCAL_MODEL_BACKENDS_GUIDE.md`
- connectors:
  - `docs/en/03_QQ_CONNECTOR_GUIDE.md`
  - `docs/en/04_LINGZHU_CONNECTOR_GUIDE.md`
  - `docs/en/10_WEIXIN_CONNECTOR_GUIDE.md`
  - `docs/en/16_TELEGRAM_CONNECTOR_GUIDE.md`
  - `docs/en/17_WHATSAPP_CONNECTOR_GUIDE.md`
  - `docs/en/18_FEISHU_CONNECTOR_GUIDE.md`
  - `docs/en/19_EXTERNAL_CONTROLLER_GUIDE.md`
- BenchStore:
  - `docs/en/22_BENCHSTORE_YAML_REFERENCE.md`
  - `docs/en/23_BENCHSTORE_GITHUB_RELEASES_SPEC.md`

Supplementary contributor notes:

- `CONTRIBUTING.md`
- `CLAUDE.md`

If a supplementary doc conflicts with this file, current code, or the maintainer docs in `docs/en/90_*` and `docs/en/91_*`, follow:

1. current code and tests
2. this `AGENTS.md`
3. `docs/en/90_ARCHITECTURE.md` and `docs/en/91_DEVELOPMENT.md`
4. supplementary contributor docs

## Scope Boundary For Coding Agents

Do not confuse repository contribution rules with the in-product DeepScientist agent contract.

Examples:

- files under `src/prompts/` and `src/skills/` often tell the runtime agent to use `bash_exec`
- that is a product/runtime behavior contract
- it is not a rule that repository contributors must avoid normal local shell commands while editing this repo

When changing prompt or skill behavior, preserve that distinction explicitly.

## Non-Negotiable Contracts

### 1. One quest = one Git repository

- Every quest has one absolute `quest_root`.
- All durable quest content stays inside that quest root.
- Branches and worktrees express divergence inside that quest repository.

### 2. Python runtime, npm launcher

- The authoritative runtime lives under `src/deepscientist/`.
- `bin/ds.js` remains a thin launcher over the Python daemon and built UI bundles.
- The public npm package publishes as `@researai/deepscientist`.
- Public npm installs must ship prebuilt `src/ui/dist/` and `src/tui/dist/` bundles.
- Do not rely on end-user `postinstall` builds for the public npm path.

### 3. Only three public built-in MCP namespaces

Keep the public built-in MCP surface limited to:

- `memory`
- `artifact`
- `bash_exec`

Git behavior belongs inside `artifact`.
Durable shell execution belongs inside `bash_exec`.
Do not add new public MCP namespaces such as `git`, `connector`, or `runtime_tool`.

### 4. Prompt-led, skill-led workflow

- The prompt defines workflow expectations and filesystem contract.
- Skills provide specialized execution behavior.
- The daemon persists, restores, and routes state, but should stay thin.
- Avoid hard-coding a large central stage scheduler when prompt plus skills are enough.

### 5. Registry-first extension points

Prefer small registries for:

- runners
- channels
- connector bridges
- skill discovery
- managed local runtime tools
- optional plugin adapters

Prefer `register_*()`, `get_*()`, and `list_*()` APIs over large dispatch branches.

### 6. Shared web and TUI contract

- The web UI and TUI must consume the same daemon API and event model.
- If an API route changes, update the daemon, web client, TUI client, and tests together.
- Preserve `/projects` and `/projects/:questId` style routing in the web workspace.

### 7. QQ is first-class, but still generic

- QQ support is part of the core product shape.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResearAI/DeepScientist](https://github.com/ResearAI/DeepScientist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
