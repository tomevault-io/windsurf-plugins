---
trigger: always_on
description: `laravel/lsp` is a Laravel Zero PHP CLI that is compiled into the Laravel LSP binary. It parses PHP and Blade code, extracts framework-aware symbols and context, and contains the Laravel LSP server.
---

# Laravel LSP

## Project

`laravel/lsp` is a Laravel Zero PHP CLI that is compiled into the Laravel LSP binary. It parses PHP and Blade code, extracts framework-aware symbols and context, and contains the Laravel LSP server.

## LSP Server

The LSP server lives in `app/Lsp/` and is invoked via `server` or `server lsp`. It runs as a long-lived process over stdio.

The goal is to provide completion, hover, diagnostic, link, and code-action behavior from the LSP server.

When working on the LSP server, do not write test plans or tests unless explicitly asked.

## Architecture

- The server communicates over stdio using JSON-RPC/LSP framing.
- `Server` owns message dispatch, lifecycle handling, request routing, and notification listeners.
- Request handlers live in `app/Lsp/Methods/`; notification listeners live in `app/Lsp/Listeners/`.
- `Project` owns initialized project URI/path state through `FileUri`, initialization options, the `ScriptRunner`, and the `ProjectIndex` for project data access.
- `Project` stores LSP `initializationOptions`; use its `InteractsWithData` helpers for feature flags and configuration values.
- `DocumentManager` tracks open editor documents, while `Document` owns cached parser-backed analysis for the current document version.
- `ProjectIndex` manages project data providers, receives the container for provider construction, and invalidates matching provider data after watched-file changes. Watched-file notifications should invalidate provider data only unless a feature explicitly needs broader refresh behavior.
- `DataProvider` implementations expose project facts such as routes and views, and own template loading, parsing, watcher patterns, changed-path matching, and cache state.
- When a project fact needs a reusable shaped view, put small helper accessors directly on the relevant `DataProvider`, such as `Auth::policies()`, so feature providers can reuse normalized data without duplicating collection shaping.
- LSP feature behavior is exposed through provider contracts in `app/Lsp/Contracts/` such as `LinkProvider`, `HoverProvider`, `DiagnosticProvider`, and `CompletionProvider`.
- `FeatureRegistry` constructs the active providers for each LSP capability. Request handlers and listeners should use the injected `FeatureRegistry` instead of directly instantiating domain providers.
- Domain features should follow the existing provider structure for their capability. Some features expose one feature class through adapters such as `LinkFeature`, `HoverFeature`, and `DiagnosticFeature`; others, such as routes, expose separate `LinkProvider`, `HoverProvider`, and `DiagnosticProvider` classes that are constructed directly by `FeatureRegistry`.
- Feature providers should stay thin when shared document mapping exists. For routes, `RouteLinkProvider`, `RouteHoverProvider`, and `RouteDiagnosticProvider` own capability-specific configuration checks, while `RouteDocumentMapper` owns route argument detection, accepted argument filtering, route lookup, Volt component lookup, and conversion to links, hovers, and diagnostics.
- Shared mapper-style features should put reusable document traversal in `app/Lsp/Features/Support/DocumentMapper`, keep domain patterns and output conversion in the domain mapper, and use `DetectedArguments` with `Pattern` so features act only on parser-detected arguments.
- Completion providers should follow the same capability-provider shape as links, hovers, and diagnostics: request handlers use `FeatureRegistry`, and feature providers own configuration gates.
- Keep feature providers thin when a mapper exists. Providers should usually check config, construct the mapper or selector, and return the capability result.
- `Document::detect()` is for completed document analysis; `Document::autocomplete($position)` is for incomplete cursor context and should parse content up to the cursor.
- Use `DetectedArguments` and `DetectedArgument` for full-document references such as links, hovers, and diagnostics.
- Use `AutocompleteArguments` and `AutocompleteArgument` for completion contexts where the target argument may not exist yet or may not have parser ranges.
- Reuse `Pattern` for both detected and autocomplete matching. Put parser-shape traversal helpers on `DetectedArgument` or `AutocompleteArgument`, not in feature providers.
- In mapper-style features, `DocumentMapper::completions()` should mirror `links()`, `hover()`, and `diagnostics()`, while `toCompletions()` remains opt-in with a default empty result unless completion support is implemented.
- Register more specific completion providers before broader ones in `FeatureRegistry::completions()` so specialized completions win first.
- Listeners should usually decide when to call providers and publish responses, not whether a feature is enabled.
- Diagnostic publishing should be document-scoped by default: publish on `textDocument/didOpen` and `textDocument/didChange`, clear on `textDocument/didClose`, and avoid publishing all open documents unless there is a concrete product requirement.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laravel/lsp](https://github.com/laravel/lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
