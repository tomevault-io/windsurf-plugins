---
trigger: always_on
description: > **This file is the source of truth.** `CLAUDE.md` is a symlink to this file.
---

# ListKit — Agent Instructions

> **This file is the source of truth.** `CLAUDE.md` is a symlink to this file.
> Agents should create and update `AGENTS.md` files (never edit `CLAUDE.md` directly — it's a symlink).
> When creating a new `AGENTS.md`, always create a corresponding `CLAUDE.md` symlink:
> `ln -s AGENTS.md CLAUDE.md`

## Project Overview

ListKit is a high-performance, type-safe UICollectionView framework for iOS, organized as two Swift modules in a single package:

- **ListKit** (`Sources/ListKit/`) — Low-level diffing engine. Drop-in replacement for `UICollectionViewDiffableDataSource` with an O(n) Heckel diff algorithm.
- **Lists** (`Sources/Lists/`) — High-level declarative API built on ListKit. Provides ViewModel-driven data sources, result-builder DSL, pre-built configurations, and SwiftUI wrappers.

Lists depends on ListKit. External consumers import either `ListKit` (low-level) or `Lists` (high-level; must also `import ListKit` separately if using its types directly).

## Architecture

```
SwiftUI Wrappers (SimpleListView, GroupedListView, OutlineListView)
        ↓
Pre-Built Configurations (SimpleList, GroupedList, OutlineList)
        ↓
Data Sources (ListDataSource, MixedListDataSource)
        ↓
Builder DSL + Protocols (CellViewModel, SnapshotBuilder, AnyItem)
        ↓
ListKit Core (Snapshot, HeckelDiff, SectionedDiff, CollectionViewDiffableDataSource)
```

## Build & Test

**Always use the Makefile** for building, testing, formatting, and other project tasks. The Makefile wraps complex `xcodebuild` invocations with the correct workspace, scheme, destination, and derived data paths. Do not invoke `xcodebuild` or `swift build` directly — use `make` targets instead.

| Command | Description |
|---|---|
| `make setup` | First-time setup (install Tuist, generate project, install hooks) |
| `make build` | Build both frameworks |
| `make test` | Run all tests |
| `make test-listkit` | Run ListKit tests only |
| `make test-lists` | Run Lists tests only |
| `make benchmark` | Run comparative benchmarks |
| `make format` | Format code with SwiftFormat |
| `make lint` | Lint code with SwiftFormat |
| `make docs` | Generate DocC documentation |
| `make open` | Open in Xcode |

Tests require an iOS Simulator destination. The Makefile defaults to `iPhone 17 Pro`.

## Code Style

- **`// ABOUTME:` comments** — Every `.swift` file must begin with a `// ABOUTME:` comment block (before imports). This is a 1-2 line summary of what the file contains. Format:
  ```swift
  // ABOUTME: Short description of this file's purpose.
  // ABOUTME: Optional second line with additional context.
  ```
  Rules:
  - Place at the very top of the file (line 1), before any `import` statements
  - Use `// ABOUTME:` prefix on each line (not `///` or `/* */`)
  - Keep each line under 100 characters
  - First line: what the file defines or provides
  - Optional second line: key relationships, constraints, or non-obvious details
  - When creating or modifying a file, ensure it has an `// ABOUTME:` comment
- **Swift 6** strict concurrency — all public types must be `Sendable`
- **SwiftFormat** with Airbnb-based config (see `.swiftformat`)
  - 2-space indentation
  - Max line width: 130 (recommend ≤100)
  - `--self remove` (no explicit `self`)
  - Trailing commas on multi-element collections only
  - `organizeDeclarations` is enabled — declaration order matters
- Run `make format` before committing
- Run `make lint` to check without modifying

## Performance

**Performance is the #1 priority for the ListKit module.** ListKit exists because Apple's implementation is too slow — every change to `Sources/ListKit/` must preserve or improve performance.

Any change that could affect performance **must**:
1. Run `make benchmark` before and after the change
2. Compare results to confirm no regression
3. Update the README with new numbers if baselines change

Do not merge changes that regress benchmark numbers without explicit approval. See [`Sources/ListKit/AGENTS.md`](Sources/ListKit/AGENTS.md) for current baselines and details.

## Changelog

**ALL pull requests MUST include a changelog entry.** The project maintains a [`CHANGELOG.md`](CHANGELOG.md) following the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

When submitting a PR:
1. Add an entry under the `## [Unreleased]` section at the top of `CHANGELOG.md` (create the section if it doesn't exist)
2. Use the appropriate subsection: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, or `Security`
3. Write a concise, user-facing description of the change
4. PRs without a changelog entry should not be merged

## Documentation

The project uses **DocC** to generate API documentation, hosted as static files in the `docs/` directory (checked into the repo). Run `make docs` to regenerate.

**You MUST run `make docs`** and commit the updated `docs/` output when any of the following change:

- **Public API** in `Sources/ListKit/` or `Sources/Lists/` — adding, removing, or renaming any `public` type, method, property, or protocol requirement
- **Doc comments** (`///`) on public symbols
- **DocC catalog content** — anything under `Sources/ListKit/ListKit.docc/` or `Sources/Lists/Lists.docc/` (articles, tutorials, media, symbol extensions)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Iron-Ham/Lists](https://github.com/Iron-Ham/Lists) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
