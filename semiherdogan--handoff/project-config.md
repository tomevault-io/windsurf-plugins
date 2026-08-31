---
trigger: always_on
description: Guidelines for AI/code agents and contributors working in this repository.
---

# AGENTS.md

Guidelines for AI/code agents and contributors working in this repository.

## Project Snapshot

- Project: `handoff`
- Type: local-first CLI tool for autonomous dev-loop prompt generation
- Runtime model: no provider/network API calls in core flow
- Language: Rust

## Core Intent

`handoff` manages a structured feature workspace under `.handoff/` and generates prompts (`start`, `spec`, `design`, `tasks`, `continue`, `drift`) for coding assistants.

The tool should remain:

- minimal
- deterministic
- local-first

## Repository Map (Where Things Live)

- `src/main.rs`: CLI entrypoint and command dispatch
- `src/cli.rs`: clap command/flag definitions
- `src/commands/`: per-command behavior (`init`, `start`, `continue`, `prompt`, `status`, etc.)
- `src/core/`: workspace, feature file handling, state parsing/guards
- `src/templates/`: template manager + prompt resolvers
- `templates/default/`: embedded default markdown templates
- `docs/`: guide and reference documentation that keep `README.md` focused on onboarding
- `.github/workflows/release.yml`: tag-triggered release pipeline

## Workspace Contract

Expected structure:

```text
.handoff/
  config.toml
  current -> features/<feature-name>
  features/
    <feature-name>/
      FEATURE.md
      SPEC.md
      DESIGN.md
      STATE.md
      SESSION.md
      DECISIONS.md
```

Artifact responsibilities:

- `config.toml`: workspace-level prompt settings such as the preferred prose language for generated prompts; missing `language` must fall back to English
- `FEATURE.md`: raw feature intent and owner constraints
- `SPEC.md`: normalized behavioral requirements and acceptance criteria
- `DESIGN.md`: technical design scaffold; may stay lightweight for simple features
- `STATE.md`: execution plan, progress markers, and execution evidence
- `SESSION.md`: continuation-safe session summary
- `DECISIONS.md`: durable feature-level product or architecture decisions

`.handoff/current/` is reserved for handoff workflow artifacts only:

- allowed files: `FEATURE.md`, `SPEC.md`, `DESIGN.md`, `STATE.md`, `SESSION.md`, `DECISIONS.md`
- do not place extra project documentation, analysis notes, reports, or drafts there
- if permanent project docs are needed, put them in normal repository locations such as `docs/`, the repository root, or the closest relevant module directory

## STATE.md Invariants (Important)

Execution plan markers:

- `[ ]` pending
- `[>]` current
- `[x]` completed

Accepted plan line forms:

- `- [ ] step`
- `* [>] step`
- `1. [x] step`

Guard behavior for `continue`:

- fail if execution plan is not initialized
- fail if multiple `[>]` exist
- fail if there are no remaining steps

Deterministic guard errors are part of the contract; do not silently relax them.

`STATE.md` parser-sensitive structure remains English-only unless the parser contract is intentionally changed:

- section headers like `# Current Step`, `# Execution Plan`, and `# Risks`
- execution markers `[ ]`, `[>]`, and `[x]`

Execution prompts should add evidence for completed steps under `# Evidence`, including changed files, validation commands, and results.

## Commands (Current Surface)

- `handoff init [feature] [--force]`
- `handoff switch <feature>`
- `handoff run [--copy] [--raw]`
- `handoff generate [--copy] [--raw]`
- `handoff start [--copy] [--raw]`
- `handoff drift [--copy] [--raw]`
- `handoff spec [--copy] [--raw]`
- `handoff design [--copy] [--raw]`
- `handoff tasks [--copy] [--raw]`
- `handoff continue [--copy] [--raw]`
- `handoff prompt [generate|start|spec|design|tasks|continue|context|drift] [--copy] [--raw]`
- `handoff status`
- `handoff next`
- `handoff validate`
- `handoff version`
- `handoff list`
- `handoff clean [--force]`
- `handoff archive <feature>`
- `handoff completion <shell>`
- `handoff upgrade`
- `handoff export [--force]`
- `handoff ignore`

## Command Intent (When to Use What)

- `init`: create/select feature workspace, optionally replace existing `.handoff/current` with `--force`
- `init`: create/select feature workspace, scan repository context readiness, and point users to `handoff prompt context` when high-value context is missing
- `generate`: generate a planning-only prompt that updates `SPEC.md`, optional `DESIGN.md`, `STATE.md`, `SESSION.md`, and durable `DECISIONS.md` entries without implementing code
- `run`: inspect the active workspace state and emit the next prompt automatically (`generate`, `start`, or `continue`)
- `start`: generate an execution-only prompt; require an existing valid execution plan before implementation begins
- `spec`: generate a prompt that turns `FEATURE.md` into `SPEC.md`
- `design`: generate a prompt that turns `FEATURE.md` + `SPEC.md` into `DESIGN.md`
- `tasks`: generate a prompt that turns `SPEC.md` (+ optional `DESIGN.md`) into the `STATE.md` execution plan
- `continue`: generate continuation prompt with STATE guard checks
- `drift`: generate an audit prompt that compares saved intent and decisions against implementation without changing code
- `prompt`: raw prompt generator (`generate`, `start`, `spec`, `design`, `tasks`, `continue`, `context`, or `drift`) without continue guard semantics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [semiherdogan/handoff](https://github.com/semiherdogan/handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
