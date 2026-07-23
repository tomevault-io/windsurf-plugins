---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this
repository.

## Project Overview

TMDb is a Swift Package for The Movie Database API, supporting iOS 16+,
macOS 13+, watchOS 9+, tvOS 16+, visionOS 1+, Linux, and Windows. Built
with Swift 6.0+ and strict concurrency.

## Knowledge Base

Durable, project-specific learnings live in [`knowledge/`](knowledge/) — read it
on demand; it is not loaded here to keep this file lean. (This is reference
knowledge; `CLAUDE.md` stays imperative.)

- [`knowledge/decisions/`](knowledge/decisions/) — **ADRs** (design decisions +
  rationale).
- [`knowledge/gotchas.md`](knowledge/gotchas.md) — quirks, tooling traps, things
  that needed a lookup.
- [`knowledge/tmdb-api-notes.md`](knowledge/tmdb-api-notes.md) — live-API
  behaviours.

**Before solving a non-trivial problem**, skim the relevant file. **After learning
something durable** (a gotcha, an API quirk, a design decision), record it there —
run `/capture-knowledge` (it runs automatically before a PR in `/deliver`). Add an
ADR for any non-obvious design decision.

## Architecture

### Service-Based Design

The library uses protocol-based services with dependency injection.
`TMDbClient` is the main facade exposing 26 service properties:

```text
TMDbClient (main facade)
├── AccountService
├── AuthenticationService
├── CertificationService
├── ChangesService
├── CollectionService
├── CompanyService
├── ConfigurationService
├── CreditService
├── DiscoverService
├── FindService
├── GenreService
├── GuestSessionService
├── KeywordService
├── ListService
├── MovieService
├── NetworkService
├── PersonService
├── ReviewService
├── SearchService
├── TrendingService
├── TVEpisodeService
├── TVEpisodeGroupService
├── TVSeasonService
├── TVSeriesService
├── WatchProviderService
└── NaturalLanguageSearchService
```

The `naturalLanguageSearch` service is **Apple-platforms only** — it is
defined in a `TMDbClient` extension gated by `#if canImport(NaturalLanguage)`,
so it is unavailable on Linux and Windows. It interprets free-text queries
on-device with a deterministic planner (a rule-based intent classifier plus
`NLTagger` person-name extraction) and, on capable devices, an optional
`FoundationModelsSearchPlanGenerator` fallback for fuzzier prompts — degrading
to a plain multi-search where neither is available.

**Key files:**

- `Sources/TMDb/TMDbClient.swift` — main public API entry point
- `Sources/TMDb/TMDbFactory.swift` — dependency injection factory
- `Sources/TMDb/Domain/Services/` — service protocols and implementations
- `Sources/TMDb/Domain/Models/` — Codable data models (~170 files)
- `Sources/TMDb/Domain/LanguageModelTools/` — FoundationModels `Tool`s
  exposing TMDb to a conversational assistant (Apple-only)

### Networking Layer

Services call into `TMDbAPIClient`, which sits **above** the `HTTPClient`.
`TMDbAPIClient` adds the `api_key` query item, builds the request, validates
the response status, and decodes the body. It then delegates the raw HTTP
transport to an `HTTPClient`, which is a decorator chain: `CacheHTTPClient`
wraps `RetryHTTPClient`, which wraps the base adapter (the user-supplied
`HTTPClient` or the default `URLSessionHTTPClientAdapter`). Both retry and
cache decorators are opt-in (see `TMDbFactory.httpClient(wrapping:...)`).

```text
Service (e.g. TMDbMovieService)
└── APIClient
    └── TMDbAPIClient            (adds api_key, validates status, decodes)
        └── HTTPClient (protocol)
            └── CacheHTTPClient          (opt-in; cache hits short-circuit)
                └── RetryHTTPClient      (opt-in; exponential backoff)
                    └── URLSessionHTTPClientAdapter  (or user-supplied client)
```

In 18.0.0 an `ErrorMappingAPIClient` decorator wraps the `APIClient` to
centralise mapping of `TMDbAPIError` into the public `TMDbError`.

### Language Model Tools (Apple-only)

`TMDbToolbox` (`Sources/TMDb/Domain/LanguageModelTools/`) wraps the services
as FoundationModels `Tool`s so TMDb can back a conversational movie assistant
through a `LanguageModelSession`. It is gated by
`#if canImport(FoundationModels) && !os(tvOS)` and annotated
`@available(iOS 26, macOS 26, visionOS 26, watchOS 27, *)`.

Eight tools are exposed: `search`, `movieDetails`, `movieCredits`,
`tvSeriesDetails`, `personFilmography`, `trending`, `watchProviders`, and
`discoverMovies`. They
are reachable from `TMDbClient` via `languageModelTools` (shorthand for
`TMDbToolbox(client:).all`) and individual `*Tool` accessors (`searchTool`,
`movieDetailsTool`, …). Each tool returns compact text whose every line leads
with the relevant TMDb `id`, letting the model chain calls — search a title,
then fetch its details or watch providers.

### Test Organization

- `Tests/TMDbTests/` — unit tests with mocks and JSON fixtures
- `Tests/TMDbIntegrationTests/` — live API tests
- Uses **Swift Testing** framework (`@Test`, `#expect`, `#require`) — not
  XCTest

## Understanding the TMDb API

### OpenAPI Specification

The complete API spec is at:
**<https://developer.themoviedb.org/openapi/tmdb-api.json>**

Use this to understand endpoints, request/response schemas, query
parameters, and authentication requirements.

### TMDb MCP Server


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamayoung/TMDb](https://github.com/adamayoung/TMDb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
