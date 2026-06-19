---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## TDD is non-negotiable (read this first)

**You MUST write a failing test before writing any production code.** This rule overrides every other instinct, including "the change is small", "it's just a one-liner", "I'll add the test after". If you catch yourself opening a file under `Sources/` before a test under `Tests/` exists and fails, stop and reverse course.

**Pre-implementation gate** — before editing anything in `Sources/`, you must have done all of the following in order:

1. Stated the user-facing behaviour in one sentence (e.g. "a version is live when state is `READY_FOR_SALE`").
2. Written a test in `Tests/` that asserts the exact expected output for that behaviour.
3. Run `swift test` (or a `--filter`'d subset) and **observed the test fail** — compile error counts as a failing test only if the assertion is the reason it can't compile (e.g. missing symbol the test names).
4. Reported the red result back to the user (one line is fine: "test X fails with: <message>").

Only after step 4 may you write code under `Sources/`. The full workflow, naming rules, and framework details are in the [Testing](#testing) section.

**If you skip the gate, you are violating the project's primary rule.** Treat this the same as committing secrets or force-pushing main.

## Commands

```bash
# Build
swift build                  # Debug build
swift build -c release       # Release build

# Test
swift test                               # All tests
swift test --filter 'AppTests'           # Tests matching a pattern
swift test --enable-code-coverage        # With coverage

# Run
swift run asc <args>
make run ARGS="apps list"
```

## Architecture

Three strict layers with a unidirectional dependency flow: `ASCCommand → Infrastructure → Domain`

```
Sources/
├── Domain/        # Pure value types, @Mockable protocols — zero I/O
├── Infrastructure/# Implements Domain protocols via appstoreconnect-swift-sdk
└── ASCCommand/    # CLI entry point, output formatting, TUI
```

### Domain Layer

All models are `public struct` + `Sendable` + `Equatable` + `Codable`. The JSON encoding is the public schema. Models with optional text fields use custom `Codable` with `encodeIfPresent` to omit nil values from JSON output.

**Design rules:**
- Every model carries its **parent ID** (e.g. `AppStoreVersion.appId`, `AppScreenshot.setId`) — the App Store Connect API doesn't return parent IDs, so Infrastructure injects them
- State enums expose **semantic booleans** (`isLive`, `isEditable`, `isPending`, `isComplete`) for agent decision-making
- All repositories and providers are `@Mockable` protocols

### Infrastructure Layer

Adapts `appstoreconnect-swift-sdk` to Domain protocols. The critical pattern: mappers always inject the parent ID from the request parameter into every mapped response object.

### ASCCommand Layer

- `ASC.swift` — `@main` entry, registers all subcommands
- `GlobalOptions.swift` — `--output` (default: json), `--pretty`, `--timeout`
- `OutputFormatter.swift` — JSON/table/markdown rendering; `formatAgentItems()` merges affordances
- `ClientProvider.swift` — factory wiring auth → authenticated repositories
- `Commands/Web/` — `asc web-server` serves the REST API; `RESTRoutes.configure` composes `*Controller` structs (Hummingbird). Every new list/read command **must** also be exposed here (see "REST exposure" below)

## Key Design Patterns

### CAEOAS (Commands As the Engine Of Application State)

CLI equivalent of REST HATEOAS. Every response includes an `affordances` field with ready-to-run CLI commands so an AI agent can navigate without knowing the command tree. Affordances are **state-aware** — e.g. `submitForReview` only appears when `isEditable == true`.

All domain models implement `AffordanceProviding`:
```swift
protocol AffordanceProviding {
    var affordances: [String: String] { get }
}
```

`OutputFormatter.formatAgentItems()` merges affordances into the encoded JSON output.

### REST exposure (every feature must ship both CLI and REST)

The `asc web-server` command exposes the same functionality as the CLI over HTTP so an agent can drive it as a REST service. **A feature is not complete until it is reachable via REST.**

Required steps when adding a new list/read command (or any command that returns data an agent might want over HTTP):

1. **Make the domain model `Presentable`** — `tableHeaders` + `tableRow`. Needed because `restFormat` is `<T: Encodable & AffordanceProviding & Presentable>`
2. **Use `structuredAffordances` (not raw `affordances`)** on the model — the REST renderer derives `_links` from `Affordance` values; returning a plain `[String: String]` leaves `_links` empty
3. **Give the command an `affordanceMode` parameter** — `func execute(repo:…, affordanceMode: AffordanceMode = .cli)` forwards to `formatter.formatAgentItems(items, affordanceMode: affordanceMode)`. The same `execute` runs for both `cli` and `rest` modes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tddworks/asc-cli](https://github.com/tddworks/asc-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
