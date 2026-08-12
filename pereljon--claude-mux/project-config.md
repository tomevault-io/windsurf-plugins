---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

**claude-mux** -- persistent Claude Code sessions in tmux. Shell script + macOS LaunchAgent. Deliverables: `claude-mux`, `install.sh`, `config.example`, `com.user.claude-mux.plist`.

This is an open-source project with external users. Treat it accordingly: safety, portability, stability matter.

## Feature Freeze

**Status: LIFTED** as of 2026-05-30. v2.0 planning and implementation work has begun. Patches in the v1.14.x range and v2.x minors are in scope.

Sequencing is tracked in `docs/ISSUES.md`:
- **Planned Patches** section: small UX work shipping as v1.14.x minors before v2.0.
- **v2.0 Milestone** section: architectural changes split across v2.0 ("Self-healing + situational awareness"), v2.1 ("Context discipline"), v2.2 ("Agent network").

**Prior exception (v1.13.0):** `--restart --fresh` / "restart this session fresh" / "kill this session" was shipped under the previous freeze due to high severity (MCP installs unusable without it).

## Design Principles

Infrastructure, not a framework. Keep sessions alive, get out of the way.

- **Lean over featureful.** Don't duplicate what Claude Code or tmux already handle.
- **Support, don't impose.** Make Claude Code persistent and accessible, not reshaped.
- **Conversational first.** Natural language in-session is the primary interface.
- **Eliminate complexity, don't relocate it.** Every abstraction must remove more burden than it introduces.
- **Session management is invisible.** Claude should be able to manage sessions without permission prompts interrupting the conversation. Achieved two ways: (1) claude-mux is added to each project's allow list by `setup_claude_mux_permissions()` so Claude can run it freely; (2) the injection instructs Claude to use claude-mux rather than raw shell commands that would trigger prompts. Destructive operations (e.g. `--delete`) may still require confirmation - that's intentional, not a gap.
- **Session names, not paths.** CLI commands operate on session names, not directory paths. The script resolves session names to directories internally via tmux (running sessions) or `PROJECT_DIRS` scanning (idle projects). Exceptions that accept paths (e.g. `--move` destination, `-d`/`-n` launch directory) require explicit approval before adding.

## Documentation Roles

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Conventions, checklists, guardrails for working in this repo |
| `dev/IMPLEMENTATION-SPEC.md` | Product spec: architecture, config reference, design decisions, translation standards, deprecation policy |
| `README.md` | Landing page: install, capabilities, conversational examples, links to docs |
| `CHANGELOG.md` | What changed per release |
| `docs/CLI.md` | Full CLI command reference for scripting and automation |
| `docs/GUIDE.md` | Configuration, session details, internals, troubleshooting |
| `docs/INSTALL.md` | Full installation guide (curl, Homebrew, manual, uninstall) |
| `docs/FAQ.md` | Common questions about claude-mux |
| `docs/ISSUES.md` | Open bugs, planned features, resolved issues |
| `dev/CODEMAP.md` | Function **purposes** (prose), config vars, dispatch table, marker file registry — for locating things in the script. The function→location index is generated (see next row) |
| `dev/CODEMAP.index.md` | **Generated** by `make codemap` — function→`module:within-module-line` index. Never hand-edit; guarded by `make check` |
| `dev/features/INDEX.md` | **Generated** by `make features-index` — the build queue projected from each feature doc's `kind:`/`lifecycle:` frontmatter. Read it after a context clear to see what's `ready` to build |
| `dev/SKELETON.md` | Pseudo-code showing script structure, logic flow, and key invariants — for understanding how the script works |
| `dev/features/<feature>.md` | Per-feature design doc: the implementable spec for a feature, extracted from `docs/ISSUES.md` once it's ready to build. MUST carry `kind:` + (for `kind: feature`) `lifecycle:` frontmatter |
| `dev/features/<feature>-tests.md` | Per-feature test plan: happy path, edge cases, verification steps, pre-build and post-build checks |

**Feature design + test docs convention (decided 2026-06-07):** when a planned feature in `docs/ISSUES.md` matures to "ready to build," lift it into a dedicated design doc at `dev/features/<feature>.md` and a matching test plan at `dev/features/<feature>-tests.md`. `docs/ISSUES.md` stays the planned-features tracker; the `dev/features/` pair is the implementable spec + test plan that the build works from. Verify assumptions the design rests on *before* finalizing the design doc, so the docs reflect verified reality, not assumptions.

**Feature-doc frontmatter (controlled vocab, enforced by `make features-index`):** every `dev/features/<feature>.md` carries `kind:` and, for `kind: feature`, a `lifecycle:`. The feature index FAILs the commit on a missing/unknown value, so these are not optional.
- **`kind:`** — `feature` (default; buildable) or `investigation` (analysis-only, no build lifecycle; excluded from the build queue, like `*-tests.md`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pereljon/claude-mux](https://github.com/pereljon/claude-mux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
