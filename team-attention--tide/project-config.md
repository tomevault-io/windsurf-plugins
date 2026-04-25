---
trigger: always_on
description: This `AGENTS.md` applies to the entire repository and serves as the repo-level instruction file for Codex.
---

# Tide - Codex Repository Instructions

This `AGENTS.md` applies to the entire repository and serves as the repo-level instruction file for Codex.

Repo-wide engineering rules live here. Tool-specific workflow wrappers stay in `.claude/commands/spets.md` and `.codex/skills/spets/SKILL.md`.

## Evidence-First (Blocking)

Every factual claim in a message MUST be backed by evidence gathered in the current conversation.
Evidence means: code you read, search results you got, docs you checked, or something the user told you.
If you have no evidence, gather it before responding or explicitly ask the user.

This is a blocking rule. Do not write a factual claim first and plan to verify it afterward. Verify first, respond second.

How to apply:
1. Before describing how something works, read the relevant code first.
2. Before claiming something exists or does not exist, search for it first.
3. Before proposing an approach, understand the current system by reading code first.
4. If no evidence can be found, say "I do not know" and ask the user.

Violations include:
- Describing architecture, threading model, data flow, or patterns without reading the code.
- Claiming a feature exists or does not exist without searching.
- Proposing technical tradeoffs based on assumptions about the codebase.
- Saying "currently X works like Y" without having read X.

This applies to everything: how code works, what a function does, whether something is used, side effects of a change, external library APIs and their behavior, and similar claims.

## Domain Language (Required)

All code, commits, PRs, and discussions MUST use the terms defined in `docs/glossary.md`.
Before writing code or describing changes, check the glossary. If a term does not exist, add it.

Key terms to always use precisely:
- Pane (not "panel", "tab", "window") - a content container with a PaneId
- PaneKind - the 5 types: Terminal, Editor, Diff, Browser, Launcher
- Workspace - an isolated set of panes + layout + focus (not "tab group", "session")
- TabGroup - multiple panes stacked in one layout slot (not "workspace")
- FocusArea - FileTree or PaneArea (not "focus mode", "focus zone")
- SplitLayout - the binary tree of splits (not "grid", "tiling")
- ModalStack - mutually-exclusive popups (not "dialog", "overlay")
- GlobalAction - a user-intent command from keybinding (not "event", "message")
- Generation - monotonic counter for cache invalidation (not "version", "revision")

## Bounded Contexts (Modules)

All code lives in tide-app (monocrate). Each module is a bounded context:

| Module | Path | Responsibility | Key Entities |
|--------|------|---------------|--------------|
| core_types | `domain/core_types.rs` | Shared types and traits | PaneId, Rect, Key, TerminalGrid |
| layout | `domain/layout/` | Binary split tree | SplitLayout, TabGroup |
| terminal | `domain/terminal/` | PTY and grid sync | Terminal |
| editor | `domain/editor/` | Text buffer and cursor | EditorState |
| input | `domain/input/` | Keybinding resolution | Router, Hotkey, GlobalAction |
| tree | `domain/tree/` | Filesystem and git status | FsTree |
| platform | `adapter/outward/platform_native/` | Native macOS windowing | PlatformEvent, PlatformWindow |
| renderer | `adapter/outward/renderer/` | GPU rendering pipeline | WgpuRenderer, GlyphAtlas |
| lsp | `adapter/outward/lsp_client/` | Language server protocol | LspClient, LspManager |

Aliases in `main.rs`: `pub(crate) use domain::terminal as tide_terminal;` and similar `crate::tide_X::` paths work everywhere.

## Feature Development (MUST)

When adding a new feature or fixing a bug, follow this order. Do not skip steps or reverse the order.

```text
1. Spec -> Understand the system -> Clarify requirements with user -> Write spec
2. Test -> Write behavior tests for each Business Rule (crates/tide-app/src/behavior_tests.rs)
3. Code -> Write code that passes the tests
```

- Never skip or reverse this order, even when told "just do it all" or "do not ask questions". Those instructions mean work autonomously, not skip the process.
- No code without a spec, and no implementation without tests.
- The same rule applies when modifying existing specs: spec change -> test change -> code change.
- When a new requirement is discovered mid-implementation, stop coding and loop back: update spec -> add test -> then code.
- Use domain terms from `docs/glossary.md` when writing specs. Add new terms to the glossary first if needed.

### Spec Format (`docs/specs/{feature}.md`)

```markdown
# Spec: {Name}

## Overview
### As-Is
### To-Be
### Approach
## Bounded Contexts
## Use Cases
## Invariants
## Tests
## Location
```

More specifically:
- As-Is: current state and problems, grounded in code.
- To-Be: target state after the change.
- Approach: step-by-step plan.
- Use Cases: actor, trigger, precondition, flow, postcondition, business rules.
- Tests: UC to BR to test function mapping table.
- Location: code location.

### Test Conventions

- Test module comment: `// Spec: docs/specs/{feature}.md`
- UC section comment: `// --- UC-N: {Name} ---`
- Each test references its BR: `// UC-N BR-M: {rule description}`
- Test name is a natural language sentence: `fn closing_last_pane_in_workspace_shows_launcher()`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-attention/tide](https://github.com/team-attention/tide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
