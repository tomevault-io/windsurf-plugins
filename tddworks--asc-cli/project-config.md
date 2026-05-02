---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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
4. **Add/extend a controller** under `Sources/ASCCommand/Commands/Web/Controllers/` — inject the repository, register a `group.get("/…")` route, parse query params via `request.uri.queryParameters` (use the same names as the CLI flags, e.g. `?state=&limit=&expired-only=&before=`), call the repository, return `try restFormat(items)`
5. **Wire the controller** in `Sources/ASCCommand/Commands/Web/RESTRoutes.swift` (construct with `factory.make…Repository(authProvider: auth)`)
6. **Advertise the resource** from `APIRoot.structuredAffordances` so `GET /api/v1` lists the new top-level resource
7. **Add a REST test** in `Tests/ASCCommandTests/Commands/Web/RESTRoutesTests.swift` — call `execute(repo:affordanceMode: .rest)` and assert the output contains `"_links"` and the resolved REST paths
8. **CLI and REST query-param names must match** — if the CLI uses `--expired-only`, the REST query param is `?expired-only=true`; both go through the same repository method

Shared helpers (all in `Sources/ASCCommand/Commands/Web/RESTRoutes.swift`):
- `restFormat(items)` — REST equivalent of `formatter.formatAgentItems(items, affordanceMode: .rest)`
- `jsonError(message, status:)` — JSON error response (lives in `Infrastructure/Web/ASCWebServer.swift`; `import Infrastructure`)

Controllers are structs with dependencies injected at init (Hummingbird pattern). Repositories are constructed once in `RESTRoutes.configure`, never per request.

### Resource Hierarchy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tddworks/asc-cli](https://github.com/tddworks/asc-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
