---
trigger: always_on
description: Notes for AI assistants working on this codebase.
---

# Agents & AI Assistant Notes

Notes for AI assistants working on this codebase.

## Code organization

**Rule**: Prefer modules with one clear responsibility and one primary reason to change. If a file is simultaneously handling transport, parsing, state management, UI coordination, and policy decisions, it is too broad and should be split.

**Rule**: Keep orchestrator files thin. Managers, hooks, and controllers may coordinate multiple subsystems, but protocol parsing, data normalization, persistence, and boundary-specific policy should live in dedicated helpers or sibling modules.

**Rule**: Extract trust-boundary logic into explicit boundary modules. Parsing and normalizing untrusted input from sockets, tmux, remotes, config files, or pane output should happen in focused files near the ingress point rather than being inlined throughout higher-level application code.

**Rule**: When local and remote paths share a concept but differ in transport, separate the shared model from the transport-specific implementation. For example, event parsing, remote transport, permission routing, and UI/session updates should each have their own seam instead of being interleaved in one large file.

**Rule**: Prefer small reusable primitives for repeated low-level patterns, especially around parsing, buffering, escaping, bounds enforcement, and sanitization. Do not duplicate subtle boundary logic across multiple sockets, streams, or parsers.

**Rule**: Do not turn `util/` into a dumping ground. Only place code there when it is genuinely cross-cutting, domain-agnostic, and likely to be reused by multiple subsystems. Otherwise keep helpers next to the domain that owns them.

**Rule**: File placement should follow ownership and trust boundaries, not convenience. tmux protocol code belongs with tmux, remote transport code with remote, agent event ingestion with agents, and UI-only transformations with app/components.

**Rule**: When a refactor extracts a new seam, also improve testability at that seam. New parsers, transports, mappers, and bounded-buffer helpers should usually gain direct unit tests rather than only being covered indirectly through larger integration flows.

**Rule**: Prefer additive extraction over invasive rewrites. When shrinking an oversized module, first carve out cohesive units with stable interfaces, then simplify the original orchestrator around those units. Avoid broad churn that mixes behavioral changes with structural cleanup unless necessary.

**Rule**: Shared abstractions must stay concrete and justified. Do not introduce generic factories, base classes, or indirection layers unless at least two real call sites already benefit and the abstraction reduces complexity instead of hiding it.

**Rule**: Keep generic infrastructure layers free of feature-specific methods. The tmux control client, event emitter, config system, and similar shared modules must expose domain-neutral primitives. If a feature needs to query tmux, compose the query from existing generic methods or add a new generic method that other features could also use. Feature-specific orchestration belongs in the feature's own hook or workflow module, not in the shared layer it consumes.

## Color handling

Honeymux must work on the Linux console (outside X/Wayland), which only supports 16 colors. The theme system uses 24-bit hex colors that get mapped to the nearest 16-color equivalent by the terminal. This causes visually distinct colors (like `theme.bgFocused` vs `theme.bgSurface`) to collapse to the same value.

**Rule**: Always set colors according to theme reference values, never by hard-coded hex/RGB, unless instructed otherwise.

**Rule**: Never rely solely on background color to indicate focus or selection state. Always pair it with a text-based indicator (prefix character like `›`, foreground color change, or both). This applies to all dropdown menus, list items, and interactive UI elements. Examples of correct focus indication: (i) focused dropdown items get a `▸` prefix and `theme.textBright` foreground; unfocused items use blank prefix, (ii) only the focused item shows `▸` — don't also mark the "active/current" item with a separate arrow, as that creates two competing indicators, (iii) button focus uses both color and a `▸` prefix (see e.g. options dialog).

## Dialogs

**Rule**: All confirmation dialogs for destructive operations should focus the cursor on the negative answer (e.g. cancel) by default, to help mitigate inadvertent positive responses.

## Display width

**Rule**: Treat rendered text layout in terminal cell width, not code-unit count. CJK/fullwidth characters may occupy 2 cells, combining marks may occupy 0, and mixed-width strings must still align correctly.

**Rule**: Any truncation, padding, centering, hit-testing, overlay placement, drag geometry, fixed-column layout, or absolute-position calculation that depends on rendered text width must use the repo's width-aware text helpers rather than raw `.length`, `slice()`, `padStart()`, or `padEnd()`.

**Rule**: This applies to both interactive and passive UI surfaces: tab bars, pane tabs, trees, dropdowns, overflow menus, overlays, headers, and dialogs. Do not assume passive displays are exempt from width-aware handling.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [honeymux/honeymux](https://github.com/honeymux/honeymux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
