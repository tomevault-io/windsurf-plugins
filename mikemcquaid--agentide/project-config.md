---
trigger: always_on
description: Most importantly: run `script/style --fix` before finishing any change,
---

# Agent Instructions for MikeMcQuaid/AgentIDE

Most importantly: run `script/style --fix` before finishing any change,
read `README.md` and `ARCHITECTURE.md` before changing anything and
update them in the same commit when behaviour they describe changes.
This repository is readme-driven: documentation leads, code follows.

Keep `README.md` focused on capabilities users care about, with one
concise sentence per feature and one sentence per Settings pane.
In `README.md`, put each sentence on its own line without wrapping.
Describe user benefits there; keep implementation details and regression
histories in `ARCHITECTURE.md` or the relevant platform notes.

AgentIDE is a native SwiftUI macOS app for running, steering and
reviewing sandboxed AI coding agents.

Write sentence-case imperative commit messages without
conventional-commit prefixes such as `feat:`, `fix:` or `chore:`.

## Commands

- `script/bootstrap`: install `Brewfile` dependencies and generate
  `AgentIDE.xcodeproj` with XcodeGen
- `script/build`: regenerate the Xcode project when sources
  changed, then build the app with xcodebuild
- `script/install`: build, then copy the app into /Applications so
  the running copy survives rebuilds
- `script/zip`: zip the Release build as
  `.build/AgentIDE-<version>.zip`
- `script/package`: sign the Release build with a Developer ID
  certificate, notarise and staple it, then remake its zip
- `script/test`: run the unit and integration tests, then on the
  host the App Intents tests through `xcodebuild` (a UI testing
  bundle on macOS 27 or later, since `AppIntentsTesting` drives the
  system's own runtime; they need `AGENTIDE_DEVELOPMENT_TEAM` set to a
  real team id, since the framework refuses a runner signed differently
  from the app, and `AGENTIDE_SKIP_INTENT_TESTS=1` leaves them out
  regardless)
- `script/analyze`: static analysis (SwiftLint analyzer and, on the
  host or CI, periphery for dead code)
- `script/style`: run all linters; `--fix` also applies safe fixes
- `script/performance-log [on|off|status]`: switch the installed
  app's performance log on or off (off by default; on, every process
  run, `gh` call and cache hit or miss lands in the shared
  `tmp/agentide/performance.log`)
- `script/attach [workspace]`: attach this terminal to the sandboxed
  herdr session, or list its workspaces when run without arguments
  (works as the host user or inside the sandbox)

## Repository Structure

- `README.md`: user workflow and features; the product specification
- `ARCHITECTURE.md`: system design, packages and data flows
- `AGENTS.md`: this file; `CLAUDE.md` is a symlink to it
- `Package.swift`, `Sources/`, `Tests/`: the Swift package targets
- `App/`: the app shell; `project.yml` defines the XcodeGen target
  (the generated `.xcodeproj` stays gitignored). `Package.swift`
  also carries these sources as `AgentIDEAppSources`, which builds
  no bundle and exists only so `swift build` type-checks them
- `bin/`: the `agentide` command shipped inside the app bundle: the
  editor shim a shell pane puts on its `PATH` (waiting only with
  `--wait`, and taking a directory anywhere inside a worktree or
  checkout to switch the window to it),
  and `agentide new`,
  which asks its way to a session over SSH the way the app does,
  defaulting to what the window last chose (published by the app as
  `agentide/session-defaults`). Nothing installs it: an SSH login
  aliases the bundled path. It speaks the way Homebrew's own scripts
  do, a blue `==>` before each step, bold labels, green defaults and
  bold `Warning:`/`Error:` labels, colour only on a terminal that has
  not set `NO_COLOR`. `CommandLineSessionTests` pins its names to
  `SessionName`
- `docs/`: the images `README.md` embeds
- `script/`: development tasks
- `Brewfile`: development dependencies
- `.github/workflows/tests.yml`: CI

## Code Standards

- Swift 6.4 with strict concurrency: App and Feature targets use
  MainActor default isolation; Domain and DataAccess are nonisolated
- Every SwiftLint and SwiftFormat rule is enabled; disable per line
  with a comment explaining why (configuration excludes only rules
  that conflict with other enabled rules or tools, with reasons)
- SwiftLint needs the full Xcode selected via xcode-select;
  CommandLineTools alone cannot load SourceKit
- UK English (organised, colour) in documentation, comments and UI
  strings; proper nouns keep their official spellings
- Name every argument a shelled command takes: `gh` and git both
  fall back to whatever is checked out or configured, and code has
  no reason to lean on the shorthand a human types. `gh pr create`
  names `--head` and `--base`, a rebase names its branch
- Keep comments minimal; prefer self-documenting code
- Two-space indentation, four-space for Swift (see `.editorconfig`)

### UI Principles

- One implementation per concern: terminals, editors, conversation
  views, markdown rendering, git access and GitHub access each have
  exactly one shared component or client. Before adding a second
  approach to any such concern, or duplicating behaviour that an
  existing surface already provides, get explicit confirmation.
- Transitions that wait on anything (resuming a session, fetching

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MikeMcQuaid/AgentIDE](https://github.com/MikeMcQuaid/AgentIDE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
