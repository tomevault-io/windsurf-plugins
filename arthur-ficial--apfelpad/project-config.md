---
trigger: always_on
description: > **apfelpad turns a plain markdown document into a spreadsheet for thinking: every span can be text, math, or an on-device AI call, authored with one unified formula syntax, rendered inline in light green, reproducible via seeds, and 100% local.**
---

# apfelpad - Project Instructions

## The Golden Goal

> **apfelpad turns a plain markdown document into a spreadsheet for thinking: every span can be text, math, or an on-device AI call, authored with one unified formula syntax, rendered inline in light green, reproducible via seeds, and 100% local.**

The full design rationale lives in `BRIEFING.md` - read it before touching code. This file is the operational playbook.

## Canonical reference repos - READ BEFORE BUILDING ANYTHING

| Repo | GitHub | Role for apfelpad |
|---|---|---|
| **apfel** | https://github.com/Arthur-Ficial/apfel | **The engine and the underlying technology.** On-device Foundation Models via the `FoundationModels` framework (macOS 26+), wrapped as a CLI + OpenAI-compatible HTTP server. apfelpad spawns `apfel --serve` on port 11450 at launch. All inference goes through apfel over HTTP. Do NOT import FoundationModels directly anywhere in apfelpad. |
| **apfel-chat** | https://github.com/Arthur-Ficial/apfel-chat | **The coding-style, TDD, release, and update-check template.** Copy the skeleton 1:1: SwiftUI `@main`, `@Observable` MVVM, protocol+mock TDD with swift-testing, `ServerManager`, `UpdateChecker`, `ChatControlServer`, SQLite raw C, `./scripts/release.sh`, Homebrew cask. Every pattern apfelpad needs is already proven in apfel-chat. **Whenever you are unsure how to do something, look at apfel-chat first.** |
| apfel-clip | https://github.com/Arthur-Ficial/apfel-clip | Color palette reference (pale green + dark green accent). |
| homebrew-tap | https://github.com/Arthur-Ficial/homebrew-tap | Where the apfelpad cask will live. |

**The two non-negotiables:**
1. **Underlying technology is apfel + FoundationModels.** Every LLM call in apfelpad is an HTTP POST to `localhost:11450/v1/chat/completions`, not a direct SDK call. This is how apfelpad gets all of apfel's hard work (context management, streaming, tool calling, retries) for free.
2. **Code style, TDD discipline, update-check process, release workflow: copied from apfel-chat.** Not "inspired by". Copied. Same file layout, same protocol+mock patterns, same `UpdateChecker` via GitHub releases API, same release script shape.

## Status

**Pre-implementation.** Only `BRIEFING.md`, `CLAUDE.md`, and `README.md` exist. No Swift code has been written yet. The first implementation pass scaffolds from `apfel-chat`.

## Language Rules

NEVER use the word "Apple" in user-visible strings. Use instead:
- "on-device" / "your Mac" / "Foundation Models on your Mac"
- "private AI" / "local AI"

Formulas are called **formulas**. Not cells, not blocks, not prompts, not AI calls. Formulas. Consistently.

## The core primitive: formulas

Every piece of computed content in apfelpad is a formula. The formula is the unit of authorship, the unit of caching, the unit of rendering, and the unit of reproducibility. If a feature does not fit into the formula model, it does not ship.

Six formulas in v1.0:

| Formula | Purpose |
|---|---|
| `=apfel(prompt, seed?)` | On-device LLM call with auto-scoped context |
| `=math(expression)` | Pure arithmetic, no LLM |
| `=ref(@anchor)` | Insert content of a named block/heading |
| `=count(@anchor?)` | Word count of doc or block |
| `=date(format?)` | Current date, optionally formatted |
| `=clip()` | Current clipboard snapshot |
| `=file(path)` | Local file content (sandboxed) |

**Auto-quoting is non-negotiable:** `=apfel(hello world)` must be canonicalized to `=apfel("hello world")` on commit. Users type English, the parser handles the quotes.

## Install & Run

(When built - currently pre-implementation.)

```bash
brew install Arthur-Ficial/tap/apfelpad
apfelpad
```

## Build from source

```bash
swift build -c release
make install
swift test                # run all tests
swift run apfelpad        # debug build
```

Requires Xcode command-line tools and `apfel` on your `PATH`.

## Architecture

1:1 clone of `apfel-chat`. Protocol-driven, TDD-first, every service has a protocol + mock.

```
Sources/
├── App/              # @main, window lifecycle, server management
│   ├── ApfelPadApp.swift
│   ├── AppDelegate.swift
│   └── ServerManager.swift          (spawns apfel --serve on 11450)
├── Models/           # Data types (Document, FormulaSpan, CacheKey, etc.)
├── Protocols/        # Service protocols (LLMService, FormulaCache, etc.)
├── Services/         # Real implementations
│   ├── FormulaParser.swift          (auto-quoting parser)
│   ├── FormulaRuntime.swift         (dispatches to per-function evaluators)
│   ├── ApfelFormulaEvaluator.swift
│   ├── MathFormulaEvaluator.swift
│   ├── ... one evaluator per formula
│   ├── ApfelHTTPService.swift       (client for apfel --serve)
│   ├── SQLiteFormulaCache.swift
│   └── MarkdownDocumentStore.swift
├── ViewModels/       # @Observable state management
│   ├── DocumentViewModel.swift
│   ├── FormulaSidebarViewModel.swift
│   └── FormulaBarViewModel.swift
└── Views/            # SwiftUI views (thin, declarative)
    ├── DocumentView.swift
    ├── FormulaSpanView.swift        (the pale-green render)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arthur-Ficial/apfelpad](https://github.com/Arthur-Ficial/apfelpad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
