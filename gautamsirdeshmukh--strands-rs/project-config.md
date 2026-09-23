---
trigger: always_on
description: This document provides guidance for AI agents working in the Strands Agents
---

# Agent Development Guide - Strands Agents for Rust

This document provides guidance for AI agents working in the Strands Agents
Rust repository. For human contributor guidelines, see
[CONTRIBUTING.md](CONTRIBUTING.md).

This file is shared by agents with different goals - writing code, opening PRs,
and helping contributors - and is organized by task.

## Context

### Repository Layout

```text
strands-rs/
├── crates/strands-agents/ # Rust SDK (Cargo)
├── tools/strandly/        # CLI tooling
├── website/               # Documentation site (Astro) - see website/AGENTS.md
├── team/                  # Governance and cross-SDK process
├── test-infra/            # Integration-test infrastructure
├── .agents/               # Agent skills and references
├── Cargo.toml             # Cargo workspace root
└── .github/workflows/     # CI (ci.yml is the merge gate)
```

Determine which part of the repository you are changing and follow its
conventions. Documentation work must also follow `website/AGENTS.md`.

### Where the "why" lives: `team/`

Before designing a feature or changing an API, read the relevant context in
`team/`. It captures the reasoning the code itself does not:

- **`team/designs/`** - RFC-style proposals for significant features (numbered
  `NNNN-*.md`). This is the richest source of architectural context: problem
  framing, the chosen approach, alternatives considered, and consequences.
- **`team/DECISIONS.md`** - lightweight architecture decision records for
  smaller calls.
- **`team/TENETS.md`** - the principles a contribution should align with.
- **`team/API_BAR_RAISING.md`** and **`team/FEATURE_LIFECYCLE.md`** - the bar
  and process for API changes and feature deprecation.

## Writing Code

- **Code conventions**: Follow the Rust patterns in
  `crates/strands-agents/`. Use `snake_case` for modules, files, functions, and
  variables and `UpperCamelCase` for types and traits.
- **Branching**: `git checkout -b agent-tasks/{ISSUE_NUMBER}`
- **Commits**: Use [conventional commits](https://www.conventionalcommits.org/)
  - `feat:`, `fix:`, `refactor:`, `docs:`, and so on.
- **CI**: `.github/workflows/ci.yml` is the merge gate and delegates to the
  Rust, package, security, and documentation workflows.
- **Skills**: Reusable repository workflows live under `.agents/skills/` - for
  PRs (`pr-create`, `pr-writer`, `pr-feedback`), docs (`docs-writer`,
  `docs-reviewer`, `docs-audit`, `docs-planner`), and code review
  (`strands-review`). See [`.agents/skills/README.md`](./.agents/skills/README.md)
  for what each does and when to use it.
- **Doc `sourceLinks` track source files**: Documentation pages under
  `website/` point at their backing implementation through `sourceLinks`
  frontmatter. When you rename or move a Rust source file, update every
  `sourceLinks` reference to the old path in the same change. The site build
  cannot detect a stale path that still parses, so search with
  `rg -n "<old/path>" website/src/content/docs`.

### Cross-SDK Conventions

The SDKs aim for parity in concepts and names, not identical code.

- **Plugin / construct naming**: Name a construct for what it does, not for the
  interface it implements. Use `AgentSkills`, `ContextOffloader`, and
  `GoalLoop`, never an `...Plugin` suffix.
- **Cross-SDK parity**: When a name, constant, or hook event exists in another
  SDK, keep it in sync.
  - Identifiers match, re-cased to Rust idiom (`camelCase` to `snake_case`;
    type names remain `UpperCamelCase`).
  - Single-word string-literal values are byte-identical (`"user"`,
    `"success"`).
  - Multi-word string-literal values use the casing required by the shared
    protocol. Convert with an explicit map; do not emit ad hoc variants.
  - Wire field names keep their wire format even when it breaks Rust casing
    conventions (`inputSchema`, `tool_use_id`). Use explicit Serde renames.
  - Hook event names are shared across SDKs, modulo language suffix
    conventions. Add the matching name when adding an event.
- **Public vs internal API**: Keep internal modules and symbols private to the
  crate. Do not make an item public merely to share it internally; use
  `pub(crate)` where appropriate.
- **Structured logging format**:
  `field=<value>, field=<value> | lowercase human-readable message`, with no
  punctuation and a pipe between multiple statements. Use structured `tracing`
  fields or Rust formatting, never printf placeholders.
- **Evergreen comments**: Comments explain what the code does and why, never
  how it changed or what it used to do. Regression tests link the issue they
  guard against and state the guaranteed behavior; tests written during
  feature development carry no issue reference.
- **Directory and file naming parity**: Use Rust `snake_case`, but preserve the
  conceptual stem word-for-word so names remain mechanically translatable
  (`conversation-manager` to `conversation_manager`,
  `vended-plugins` to `vended_plugins`).

### Rust Quality Rules

- Preserve provider wire names explicitly with typed adapters and Serde
  attributes.
- Translate provider failures into the SDK's typed errors and preserve the
  original source where the error contract permits it.
- Keep async streams incremental; do not buffer complete model responses.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gautamsirdeshmukh/strands-rs](https://github.com/gautamsirdeshmukh/strands-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
