---
trigger: always_on
description: **Every factual claim in a response MUST be backed by evidence you gathered in this conversation.**
---

# Tide — Project Rules

## Evidence-First (BLOCKING)

**Every factual claim in a response MUST be backed by evidence you gathered in this conversation.**
Evidence means: code you read, search results you got, docs you checked, or something the user told you.
If you have no evidence, you MUST gather it before responding — or explicitly ask the user.

**This is a blocking rule.** Do NOT write a response containing factual claims, then plan to verify afterward. Verify FIRST, respond SECOND.

How to apply:
1. **Before describing how something works** — read the relevant code first
2. **Before claiming something exists or doesn't exist** — search for it first
3. **Before proposing an approach** — understand the current system by reading code first
4. **If no evidence can be found** — say "I don't know" and ask the user

Violations include:
- Describing architecture, threading model, data flow, or patterns without reading the code
- Claiming a feature exists or doesn't exist without searching
- Proposing technical tradeoffs based on assumptions about the codebase
- Saying "currently X works like Y" without having read X

This applies to everything: how code works, what a function does, whether something is used, side effects of a change, external library APIs and their behavior, etc.

## Domain Language (Required)

All code, commits, PRs, and discussions MUST use the terms defined in `docs/glossary.md`.
Before writing code or describing changes, check the glossary. If a term doesn't exist, add it.

Key terms to always use precisely:
- **Pane** (not "panel", "tab", "window") — a content container with a PaneId
- **PaneKind** — the 5 types: Terminal, Editor, Diff, Browser, Launcher
- **Workspace** — an isolated set of panes + layout + focus (not "tab group", "session")
- **TabGroup** — multiple panes stacked in one layout slot (not "workspace")
- **FocusArea** — FileTree or PaneArea (not "focus mode", "focus zone")
- **SplitLayout** — the binary tree of splits (not "grid", "tiling")
- **ModalStack** — mutually-exclusive popups (not "dialog", "overlay")
- **GlobalAction** — a user-intent command from keybinding (not "event", "message")
- **Generation** — monotonic counter for cache invalidation (not "version", "revision")

## Bounded Contexts (Modules)

All code lives in tide-app (monocrate). Each module is a bounded context:

| Module | Path | Responsibility | Key Entities |
|--------|------|---------------|--------------|
| core_types | `domain/core_types.rs` | Shared types & traits | PaneId, Rect, Key, TerminalGrid |
| layout | `domain/layout/` | Binary split tree | SplitLayout, TabGroup |
| terminal | `domain/terminal/` | PTY & grid sync | Terminal |
| editor | `domain/editor/` | Text buffer & cursor | EditorState |
| input | `domain/input/` | Keybinding resolution | Router, Hotkey, GlobalAction |
| tree | `domain/tree/` | Filesystem & git status | FsTree |
| pane | `domain/pane/` | Per-PaneKind state | TerminalPane, EditorPane, BrowserPane, DiffPane |
| modal | `domain/modal/` | ModalStack popups | FileFinderState, GitSwitcherState, ContextMenuState |
| state | `domain/state/` | Grouped App state | WorkspaceManager, GatewayStatus, BackgroundServices |
| services | `application/services/` | Use-case orchestration on App | action, pane_create, file_tree, session, workspace |
| gateway (cli) | `adapter/inward/cli_adapter/` | Agent Gateway: MCP/CLI dispatch over the Unix socket | GatewayServer, CliPorts |
| platform | `adapter/outward/platform_adapter/` (macos/) | Native macOS windowing | PlatformEvent, PlatformWindow |
| renderer | `adapter/outward/renderer_adapter/` | GPU rendering pipeline | WgpuRenderer, GlyphAtlas |
| lsp | `adapter/outward/lsp_adapter/` | Language server protocol | LspClient, LspManager |

Aliases in `main.rs`: `pub(crate) use domain::terminal as tide_terminal;` etc. — `crate::tide_X::` paths work everywhere.

## Feature Development (MUST)

When adding a new feature or fixing a bug, follow this order. **Do not skip steps or reverse the order.**

```
1. Spec   → Understand the system → Clarify requirements with user → Write spec
2. Test   → Write behavior tests for each Business Rule (crates/tide-app/src/application/behavior_tests/)
3. Code   → Write code that passes the tests
```

- Never skip or reverse this order — even when told "just do it all" or "don't ask questions". Those instructions mean "work autonomously", NOT "skip the process".
- No code without a spec, no implementation without tests
- Same applies when modifying existing specs: spec change → test change → code change
- When a new requirement is discovered mid-implementation, STOP coding and loop back: update spec → add test → then code
- Use domain terms from `docs/glossary.md` when writing specs. Add new terms to glossary first if needed.

### Spec Format (`docs/specs/{feature}.md`)

```markdown
# Spec: {Name}

## Overview
### As-Is             ← Current state and problems (concrete, code-based)
### To-Be             ← Target state after changes
### Approach          ← Step-by-step plan to get there
## Bounded Contexts    ← Related crates
## Use Cases           ← Actor, Trigger, Precondition, Flow, Postcondition, Business Rules
## Invariants          ← Invariants that must hold

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-attention/tide](https://github.com/team-attention/tide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
