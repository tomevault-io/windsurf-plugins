---
trigger: always_on
description: - Read this `AGENTS.md` in full before making any changes to the repository; it
---

# Project Guidelines

## Notes for automated/codegen agents

- Read this `AGENTS.md` in full before making any changes to the repository; it
  is the canonical source of project conventions
- No repository-level AI instruction files were found (e.g.,
  `.github/copilot-instructions.md`, `AGENT.md`, `CLAUDE.md`, or
  `.cursor`/`.cursorrules` files); rely on this document & the concrete files
  listed below
- Quick entry points (use these scripts rather than invoking swift directly when
  possible):
  - `Scripts/bootstrap`
  - `Scripts/format`
  - `Scripts/lint -AP` (quick) / `Scripts/lint` (full)
  - `Scripts/test`
  - `Scripts/build` / `Scripts/build '' -c release`
- Useful code locations & examples (look here for patterns to follow):
  - CLI commands: `Sources/mas/Commands/` (e.g. `Install.swift`, `List.swift`)
  - Models: `Sources/mas/Models/` (e.g. `AppID.swift`, `CatalogApp.swift`)
  - Utilities: `Sources/mas/Utilities/` (e.g. `Output/Printer.swift`)
  - Tests & test naming: `Tests/MASTests/` (see `MASTests+*.swift` files)
  - Private framework headers: `Sources/PrivateFrameworks/include/CommerceKit/`
    & `Sources/PrivateFrameworks/include/StoreFoundation/` (used via the
    `PrivateFrameworks` target)
  - Build plugin: `Plugins/MASBuildToolPlugin/MASBuildToolPlugin.swift`
  - Completion scripts: `contrib/completion/` (bash/fish)
  - Packaged runtime: `libexec/bin/mas` (used by `Scripts/mas` wrapper)
- Edits & commits: preserve formatting & style (tabs, max line length, single
  newline at EOF).
- Before committing automated edits: run `Scripts/format` until it no longer
  changes files, then `Scripts/lint` & fix violations.

## Code Refactoring Guidelines

Do NOT refactor code if doing so makes the caller interface worse. Specifically:

- **Inline a utility function or computed var at a call site iff it is
  single-use**. Inlining multi-use functions or computed vars increases
  verbosity, introduces duplication bugs & makes code harder to maintain. Keep
  clean abstractions. Example of what NOT to do:
  ```swift
  // ❌ BAD: Inlining uppercasingFirst at multiple call sites
  action1.performing.prefix(1).uppercased() + action.performing.dropFirst()
  action2.performing.prefix(1).uppercased() + action.performing.dropFirst()
  // ✅ GOOD: Use the utility function
  action1.performing.uppercasingFirst
  action2.performing.uppercasingFirst
  ```
- **Never replace a clean, readable abstraction with a verbose closure**. e.g.,
  if a custom `SortComparator` or similar is used multiple times, keep it.
  Consider inlining only if the abstraction is used in exactly one place.
  Example of what NOT to do:
  ```swift
  // ❌ BAD: Replacing a clean comparator with verbose closure
  [6, 9, 3].sorted { $0.compare($1, options: .numeric) == .orderedAscending }
  [2, 8, 4].sorted { $0.compare($1, options: .numeric) == .orderedAscending }
  // ✅ GOOD: Keep the abstraction
  [6, 9, 3].sorted(using: NumericStringComparator.forward)
  [2, 8, 4].sorted(using: NumericStringComparator.forward)
  ```
- **Replace a utility call** only when the new calling interface is at least as
  simple as the current calling interface
- **Replace a utility implementation** when the new implementation is more
  correct, performant, and/or simpler than the existing implementation, in
  descending order of priority

## Minimum Versions

- **Swift:** [6.2](.swift-version)
- **Xcode:** [26](.xcode-version)
- **macOS:** [13](Package.swift)

## Distributions

1. [Homebrew Core](https://formulae.brew.sh/formula/mas) (`brew install mas` for
   macOS 14+)
2. [Homebrew custom tap formula](https://github.com/mas-cli/homebrew-tap)
   (`brew install mas-cli/tap/mas` for macOS 13+)
3. [GitHub Releases](https://github.com/mas-cli/mas/releases)
4. [MacPorts](https://ports.macports.org/port/mas/)
5. [Nix](https://mynixos.com/nixpkgs/package/mas)

## Development Workflows

### Bootstrap Development Tools

```shell
Scripts/bootstrap
```

### Lint (quick)

```shell
Scripts/lint -AP
```

### Lint (slow, includes unused code checks)

```shell
Scripts/lint
```

### Format

```shell
Scripts/format
```

### Build (debug)

```shell
Scripts/build
```

### Build (release)

```shell
Scripts/build '' -c release
```

### Test

```shell
Scripts/test
```

### Package

```shell
Scripts/package
```

## Git Workflow

- **Trunk-based development:** `main` is the trunk
- **Topic branches:** Branch from `main` (e.g., `git checkout -b feature main`)
- **Commit messages:** Follow [commit message conventions](
    https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
  )
- **Release tags:** Format like `v1.2.3`
- **Before committing:** Run `Scripts/format` repeatedly until it no longer
  modifies any files, then run `Scripts/lint`, then fix all violations

## Content Formatting

- **Newlines:** UNIX (i.e. `\n`)
- **Indentation:** Tabs (width: 2)
- **Max line length:** 120 characters
- **Unnecessary trailing whitespace:** Remove
- **File ends:** Single newline

## Content Preservation

Unless absolutely necessary for functionality or fixes, do not:

- reformat
- rename
- reorder
- respace
- reword
- remove comments

## Scripting

### Zsh

All scripting must be written in zsh, except completion scripts (which are
written in their target shell).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mas-cli/mas](https://github.com/mas-cli/mas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
