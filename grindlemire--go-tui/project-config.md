---
trigger: always_on
description: A declarative terminal UI framework for Go with templ-like syntax and flexbox layout.
---

# go-tui Project Guidelines

A declarative terminal UI framework for Go with templ-like syntax and flexbox layout.

## Git Commits IMPORTANT

Use `gcommit -m ""` for all commits to ensure proper signing.

ONLY EVER COMMIT USING THIS APPROACH

All commit messages MUST use conventional commit format. This is required for
automated releases via release-please.

Format: `<type>: <description>` or `<type>(<scope>): <description>`

| Prefix | Version Bump | Use When |
|--------|-------------|----------|
| `feat:` | minor (0.1.0 → 0.2.0) | Adding new functionality |
| `fix:` | patch (0.1.0 → 0.1.1) | Fixing a bug |
| `perf:` | patch | Performance improvements |
| `refactor:` | patch | Code changes that don't add features or fix bugs |
| `docs:` | patch | Documentation only |
| `test:` | patch | Adding or updating tests |
| `chore:` | patch | Maintenance, dependencies, tooling |
| `ci:` | patch | CI/CD changes |
| `build:` | patch | Build system changes |
| `revert:` | patch | Reverting a previous commit |

For BREAKING CHANGES (major bump, e.g. 0.1.0 → 1.0.0), add `!` after the type:
`feat!: remove deprecated API` or include `BREAKING CHANGE:` in the commit body.

## Pull Requests IMPORTANT

PR titles MUST pass the GitHub Actions checks before merging. There are two CI
workflows that run on every PR:

1. **PR Title** (`pr-title.yml`): Enforces conventional commit format on the PR
   title using `action-semantic-pull-request`. The title must start with one of
   the allowed types (`feat`, `fix`, `docs`, `chore`, `ci`, `test`, `refactor`,
   `perf`, `build`, `revert`). This is the same format as commit messages.
2. **CI** (`ci.yml`): Runs `go test ./...` on ubuntu-latest.

Both checks must pass. Always verify your PR title matches the conventional
commit format and that tests pass before considering a PR ready.

Examples:
```
gcommit -m "feat: add table element support"
gcommit -m "fix(layout): correct flexbox gap calculation"
gcommit -m "feat!: change State API to require type parameter"
gcommit -m "chore: update golang.org/x dependencies"
```

## Project Overview

go-tui allows defining UIs in `.gsx` files that compile to type-safe Go code. The framework provides:

- Declarative component syntax (similar to templ/JSX)
- Pure Go flexbox layout engine (no CGO)
- Minimal external dependencies (golang.org/x/{mod,sync,sys,tools})
- Composable widget system
- Reactive state management with generic `State[T]`
- Language server, formatter, and tree-sitter grammar for editor support

## Architecture

```
.gsx files (declarative syntax)
        │ tui generate (internal/tuigen)
        ▼
Generated Go code (*_gsx.go)
        │ imports tui "github.com/grindlemire/go-tui"
        ▼
Widget Tree + Layout Engine (internal/layout)
        │
        ▼
Character Buffer (double-buffered 2D grid)
        │
        ▼
Terminal (ANSI escape sequences)
```

All public API types live in the root `tui` package. Internal packages (`internal/layout`,
`internal/tuigen`, `internal/formatter`, `internal/lsp`, `internal/debug`) are not importable
by external consumers.

## Where to Look (By Task)

Use this section to quickly find the right files for a given change.

### Changing the public API (Element options, App config, types)

- `element.go` — Element struct definition, TextAlign/ScrollMode/OverflowMode enums
- `element_options.go` — Option funcs: WithWidth, WithHeight, WithFlexGrow, WithDirection, WithBorder, WithScrollable, WithTruncate, WithHidden, WithOverflow, WithTextGradient, WithBackgroundGradient, WithBorderGradient, etc.
- `element_options_auto.go` — WithWidthAuto(), WithHeightAuto()
- `element_accessors.go` — Getters/setters: SetText, SetBorder, SetStyle, Background, etc.
- `element_tree.go` — Tree manipulation: AddChild, RemoveChild, RemoveAllChildren
- `element_scroll.go` — Scroll methods: ScrollTo, ScrollOffset, MaxScroll, ViewportSize
- `app_options.go` — AppOption funcs: WithFrameRate, WithMouseEnabled, WithInlineHeight, WithGlobalKeyHandler, WithInputLatency, WithEventQueueSize, etc.
- `layout.go` — Re-exports from internal/layout (Direction, Justify, Align, Value, etc.)
- `color.go` — Color type, ANSIColor(), RGBColor(), HexColor(), Gradient, GradientDirection
- `style.go` — Style type with chainable methods (Bold, Dim, Italic, Foreground, etc.)
- `ref.go` — Ref, RefList, RefMap[K] for element references
- `click.go` — Click(), HandleClicks() for ref-based mouse hit testing
- `keymap.go` — KeyMap, KeyBinding, KeyPattern, On(), OnStop(), Rune(), etc.
- `component.go` — Component, KeyListener, MouseListener, Initializer, WatcherProvider interfaces
- `events.go` — Events[T] generic event bus for cross-component communication
- `mount.go` — Component caching and lifecycle (Mount, PropsUpdater)
- `modal.go` -- Modal struct, Render, KeyMap (Escape/Tab/Enter/catch-all), HandleMouse (backdrop + onActivate click)
- `modal_options.go` -- ModalOption funcs: WithModalOpen, WithModalBackdrop, WithModalCloseOnEscape, WithModalCloseOnBackdropClick, WithModalTrapFocus, WithModalElementOptions
- `app_overlay.go` -- Overlay registration: overlayEntry, registerOverlay, clearOverlays

### Changing layout behavior (flexbox algorithm)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grindlemire/go-tui](https://github.com/grindlemire/go-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
