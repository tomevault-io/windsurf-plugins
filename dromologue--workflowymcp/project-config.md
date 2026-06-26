---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Read This First — Project Constitution + Principle Docs

This project is governed by a constitution and four detail-principle documents. **Every task in this repository — from a one-line fix to a multi-file refactor — must be bracketed by reading the relevant docs at the start and verifying against them at the end.** The reading is not optional; it is the project's enforcement mechanism for consistency across contributors (human or AI).

### The five canonical documents

| Document | When to consult |
| --- | --- |
| [`specs/constitution.md`](specs/constitution.md) | Always, before starting any non-trivial task. Eight Core Principles, the Definition of Done checklist, and the Conflict-Resolution Hierarchy. The law. |
| [`specs/principles-architecture.md`](specs/principles-architecture.md) | When the task touches module boundaries, the lift catalogue (workflows.rs / aggregation.rs / audit.rs), the helper inventory, or any pin-tested invariant. Read the relevant section before editing. |
| [`specs/principles-development.md`](specs/principles-development.md) | When the task involves naming, error handling shape, function-size judgements, helper extraction, or code style. The Rust idioms section is the canonical pattern catalogue. |
| [`specs/principles-mcp.md`](specs/principles-mcp.md) | When the task adds or modifies a tool, changes wire-surface behaviour, touches pagination/caps, modifies error envelopes, or affects the dry-run / cancel / timeout machinery. |
| [`specs/principles-security.md`](specs/principles-security.md) | When the task touches authentication, secrets, logging, error messages exposed to users, audit-trail emission, or any external input validation. |

### Task lifecycle — every task, every time

The following ritual is **mandatory** for every task that ships a commit. Skipping any step is a defect — call it out in the response rather than silently omitting it.

1. **At task start** — open [`specs/constitution.md`](specs/constitution.md) and skim the eight Core Principles + the Definition of Done. If the task touches a specific concern, open the matching `principles-*.md` file from the table above and read the relevant section before writing any code. State briefly which principle docs you consulted.
2. **During the work** — when a non-obvious design choice arises, check whether one of the principles or the Helper-First Construction table already answers it. Reach for the existing helper, not a new one. When two principles conflict, apply the Conflict-Resolution Hierarchy from the constitution.
3. **Before committing** — walk the Definition of Done checklist in the constitution. Every item must be satisfied; a commit that fails any item is not done. The checklist is the merge gate.
4. **In the response that closes the task** — state explicitly which principles were honoured and (if relevant) which pin tests were updated or added. The mention is the audit trail; it makes drift visible.

### Constitution wins on conflict

When this file (CLAUDE.md) and the constitution disagree, **the constitution wins**. CLAUDE.md is the operational guide — commands, project structure, known limitations, content-formatting conventions. The constitution and principle docs are the law: design rules, helper-routing invariants, Definition of Done, security constraints. Update the constitution first when a new rule is established; let CLAUDE.md catch up to operational consequences afterwards.

## Commands

```bash
cargo build                  # compile (debug)
cargo build --release        # compile (optimized, LTO)
cargo test --lib             # run all unit tests
cargo test                   # run all tests (unit + integration)
cargo run --bin workflowy-mcp-server  # start MCP server (stdio / Claude Desktop)
cargo check                  # type-check without building

# Skill bundling (claude.ai upload)
scripts/bundle-skill.sh                      # bundle templates/skills/wflow → dist/wflow.skill.zip (PUBLIC, generic)
scripts/bundle-skill.sh --src <dir>          # alternate skill source (e.g. ~/.claude/skills/wflow for personal)
scripts/bundle-skill.sh --out <zip>          # alternate output path
```

The bundler validates frontmatter (no `<` or `>` characters, description ≤ 1024 chars) and writes the skill source as a `<skill-name>/` directory inside the zip — the structure claude.ai's Settings → Skills upload expects.

### Public vs personal bundle split

The repository ships a **public, generic** skill bundle at `dist/wflow.skill.zip` (sourced from `templates/skills/wflow/`). It carries no user-specific node IDs, no personalised pillars, no cached Workflowy data — anyone can download or fork the repo and upload this bundle to their own claude.ai account, then run the in-skill Bootstrap to point it at their own `$SECONDBRAIN_DIR/memory/` and Workflowy account. This is the only artefact the repo ships.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dromologue/workflowyMCP](https://github.com/dromologue/workflowyMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
