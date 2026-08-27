---
trigger: always_on
description: Rules for every agent and contributor working in this repository. They are binding. When a rule here conflicts with a default behavior or a global instruction, this file wins.
---

# AGENTS.md

Rules for every agent and contributor working in this repository. They are binding. When a rule here conflicts with a default behavior or a global instruction, this file wins.

## Project

AnySSH is an iOS 26 SSH client: a real terminal, remote git, remote files, and tmux/herdr multiplexer support. No backend, no account, nothing installed on the host. Keys and passphrases live in the iOS Keychain on device.

This repository is a ground-up rebuild of a working legacy codebase. Functionality is ported, not reinvented: the legacy app behaves correctly, and the goal of the rebuild is visual consistency and code quality. Treat behavior changes as bugs unless a plan phase explicitly calls for one.

## Golden Rules

- **Max 300 lines per Swift file.** Split along an existing seam (what a screen draws vs what it does, a parser's grammar vs its decoding) before the count forces an arbitrary cut. Exempt: test files, documentation, README, and asset-like files (SVG path data, generated icons, images, fixtures).
- **Zero comments.** No exceptions for explaining code. The only tolerated occurrence is a tooling directive that cannot be avoided (`swift-format-ignore`, `#warning` required by a build rule), and it must state its reason on the same line. Do not carry comments over from legacy sources when porting.
- **Single responsibility.** Every file, type, and function does one thing. If describing it needs "and", split it.
- **SOLID by default.** Protocols (ports) declare seams, adapters implement them, the composition root wires them. Depend on abstractions, inject at the root.
- **Reuse before creating.** Before writing any UI element, check `AnySSHUI/Components/`. If a variation seems better than the existing component, improve the shared component for every call site instead of forking a local copy.
- **No dead code.** A view, component, or symbol nothing reaches does not land. During the rebuild, unreachable legacy code is deleted rather than ported and logged in `docs/dead-code-report.md`.
- **Name what repeats.** A literal that appears more than twice (string, number, duration, key, layout value) becomes a named constant owned by one module: UI values in `Theme.Space`/`Theme.Motion`, accessibility ids in `UIIdentifier`, everything else in a constants enum next to its owner. Two files declaring the same magic value is a bug.
- **Match the codebase.** Follow existing naming, structure, and idiom before introducing new ones.
- **Preserve behavior.** Gestures, toolbar wiring, and the keyboard accessory are fragile and were paid for in bugs. Port their logic exactly; restyle only their appearance.
- **One view at a time.** UI work lands view by view: build the view, screenshot it, review it against this file, then start the next. Parallel work is fine inside a single view (model, subviews, tests) or while the previous view sits in review, never as a broad sweep across many screens at once.

## Repository Layout

```
AnySSH/                 app target: composition root, wiring, launch scenarios
AnySSHWidgets/          widget and Live Activity extension
AnySSHUITests/          XCUITest flows
Shared/                 types compiled into both the app and the extension
Packages/AnySSHKit/     all modules, tests, and fixtures
Config/                 xcconfig build settings and Info.plists
Scripts/                build and vendoring automation (minimal set only)
docs/                   plans and working notes, untracked
```

Module map inside `Packages/AnySSHKit/Sources/`:

```
AnySSHCore          ports + Sendable value types, imports nothing from the project
SSHTransport        libssh2 session actor, auth, host keys, PTY, exec channels
TerminalEmulator    engine protocol, output pump, input encoding
Highlighting        tree-sitter parsers, per-line token slicing
GitClient           hardened invocation and parsers
FileTransfer        blob fetch, size guards
Sessions            session registry, reconnect policy, scrollback budget
Multiplexers        tmux and herdr adapters
AnySSHMocks         conforms to every port in AnySSHCore, imports only AnySSHCore
AnySSHUI            SwiftUI + UIKit, the only target that imports SwiftTerm
```

Rules the structure enforces:

- Protocols live in `AnySSHCore`, except view-local ones under `AnySSHUI`.
- An adapter never imports another adapter. `GitClient` takes a `RemoteCommandRunner` by injection rather than importing `SSHTransport`.
- SwiftTerm may only be imported under `AnySSHUI/Terminal/SwiftTerm/`.
- Inside `AnySSHUI`: `Components/` holds every reusable control; feature folders (`Remotes/`, `Terminal/`, `Git/`, `Files/`, `Sessions/`, `Settings/`) hold implementations that compose those components. New files go where the structure says they belong, never at a root "for now".

## Component Reuse

The component catalog is the single source of visual truth. A screen is composition, not invention.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patricio0312rev/anyssh](https://github.com/patricio0312rev/anyssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
