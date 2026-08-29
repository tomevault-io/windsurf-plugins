---
trigger: always_on
description: A native macOS menu-bar launcher — a minimal Raycast: fuzzy app launcher, global and per-app hotkeys, a
---

# Tinycast

A native macOS menu-bar launcher — a minimal Raycast: fuzzy app launcher, global and per-app hotkeys, a
text/image clipboard history, an inline calculator, a floating note, snippets, quicklinks, window
management and an emoji picker. It also **runs Raycast extensions** natively, in JavaScriptCore.
SwiftUI + AppKit, running as an accessory with no Dock icon (`LSUIElement`). Zero third-party
dependencies.

## Posture: latest-only, always

**Tinycast targets one macOS — the current stable release — and nothing else.** macOS 26+, the Xcode 26
toolchain, Swift 6 language mode. There is no compatibility floor to defend, no shim layer and no
deprecation debt, and that is the single largest reason the codebase stays as small as it does.

Write code as if the platform released yesterday:

- **Prefer the modern Apple API**, always. Observation over `ObservableObject`. Swift Concurrency over
  `DispatchQueue` or completion handlers. `SMAppService` over login-item shims. Structured concurrency
  over detached bookkeeping.
- **Migrate, never wrap.** When an API gains a modern replacement, adopt it and delete the old call
  site. A wrapper that preserves an old spelling is the thing this project has spent the most effort
  removing.
- **A deprecated API is a defect**, not a warning to live with.
- **No compatibility layers, no legacy workarounds, no older architectural patterns.** Delete rather
  than deprecate; raising the minimum macOS *deletes* the code that supported the old one.
- **Never introduce backwards compatibility unless explicitly asked for it.** No version flags, no
  migration scaffolding, no "just in case" fallbacks. The codebase carries no migration, and adding
  one needs an explicit task saying so.

Carbon is a deliberate capability-gap dependency rather than inertia: nothing modern registers a
system-wide chord, and HIToolbox's TIS APIs remain the public input-source mechanism. Full reasoning in
[standards.md](docs/standards.md#posture).

## Where things are

| Folder | Holds |
| --- | --- |
| `Tinycast/App/` | `@main`, `AppDelegate`, `AppCore` — the composition root |
| `Tinycast/DesignSystem/` | shared visual primitives; `Theme.swift` is the only design-token source |
| `Tinycast/Platform/` | system shims: `Permissions`, `AppPaths`, `Signposts`, `NotificationToken`, … |
| `Tinycast/Palette/` | the palette shell: panel, window controller, `RootPaletteView`, `PaletteScreen` |
| `Tinycast/Windows/` | the non-palette AppKit surfaces: `Dialog/`, `HUD/`, `About/`, `AppWindowController` |
| `Tinycast/Features/` | one folder per feature; larger ones split `Model/` `Service/` `UI/` `Settings/` |
| `Tests/` | the standalone harnesses — one Swift file each, no XCTest target |
| `Scripts/` | every executable script: test runner, data generators, packaging, linting, editor setup |

| Read it before you | Doc |
| --- | --- |
| change how anything is wired or owned | [architecture.md](docs/architecture.md) |
| write Swift — naming, style, concurrency, budgets, comments | [standards.md](docs/standards.md) |
| claim a change is done | [testing.md](docs/testing.md) |
| build, run or regenerate data | [development.md](docs/development.md) |
| add or restyle any view | [ui.md](docs/ui.md) |
| touch one feature's internals | [features/](docs/features/) — each opens with its invariants |
| package or ship a build | [release.md](docs/release.md) |

## Non-negotiables

Never break these without an explicit task to do so. Anything feature-specific lives in that
feature's doc, under its own `## Invariants`.

- **`AppCore` is the sole owner.** New long-lived state goes on `AppCore`, wired in `start()` — never a
  competing singleton. Views reach a feature's **coordinator** through `@Environment`, not `AppCore`.
- **A file under `Features/*/Model/` may not import AppKit or SwiftUI**, and takes every environment
  fact — clock, filesystem, home directory, rates — as an injected parameter. The harnesses compile the
  shipped sources, so this is enforced by compilation rather than convention.
- **Swift 6 language mode: data-race violations are hard errors.** `@MainActor` is the default,
  cross-actor model types are `Sendable`, and heavy or IO-bound work goes off-main as `nonisolated`
  functions driven by `Task.detached`. Do not add a second actor.
- **Dark is the baseline, and a colour's dark branch is the literal it always was.** `Theme.Colors`
  resolves per appearance through `ramp`/`adaptive`; every dark value is the `Color.white.opacity(…)`
  the forced-dark build shipped, restated rather than re-derived. Retune a light branch freely — change
  a dark one only when the task is to change Dark. `AppAppearance` drives `NSApp.appearance`, and
  `.system` maps to `nil` so AppKit follows macOS on its own.
- **Tinycast presents its own dialogs — never `NSAlert`, `NSSlider` or a system popover.** A question
  goes through `DialogController`, a report through a HUD via `HUDPresenter`.
- **A networked feature fetches on a private `.ephemeral`, `urlCache = nil` session**, never
  `URLSession.shared`, so its own cache file stays the only copy on disk. `CurrencyRateStore` is the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abue-ammar/tinycast](https://github.com/abue-ammar/tinycast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
