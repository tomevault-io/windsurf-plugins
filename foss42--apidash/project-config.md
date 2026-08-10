---
trigger: always_on
description: This is the operating guide for agents working in the API Dash repository. Treat it as a working contract, not a static note. If you change behavior, architecture, commands, generated models, storage, package boundaries, or test strategy, update this file and `architecture.md` in the same change.
---

# API Dash Agent Guide

This is the operating guide for agents working in the API Dash repository. Treat it as a working contract, not a static note. If you change behavior, architecture, commands, generated models, storage, package boundaries, or test strategy, update this file and `architecture.md` in the same change.

Scope note: this guide describes the `main` branch architecture. If you are on a feature branch, verify whether the feature is merged before documenting it here. Do not describe branch-only work as mainline behavior.

## First Rules

1. Preserve user work. Check `git status --short` before edits and do not revert unrelated modified or untracked files.
2. Read before changing. Inspect the touched source, nearby tests, root docs, and relevant developer docs before implementing.
3. Prefer the existing architecture. API Dash is a Flutter/Melos monorepo using Riverpod, Hive, Freezed/json_serializable, Jinja templates, and modular packages. Do not introduce another state, storage, routing, networking, or codegen pattern unless the task explicitly requires it.
4. Keep root docs current. Working agents must update `AGENTS.md` and `architecture.md` whenever their changes alter how future agents should understand, test, or extend the system.
5. Keep mainline docs honest. GSoC proposals, local branch files, and unmerged experiments are useful references, but they are not source-of-truth architecture for these root docs.
6. Generated Dart files are build outputs. Edit source model files, then run generation. Do not manually patch `.freezed.dart` or `.g.dart` as the primary fix.
7. Tests should be focused and offline by default. Use in-memory mocks and provider overrides unless the production component specifically requires a scoped loopback server, such as OAuth callback handling.

## Mainline Feature Baseline

The current `main` branch supports:

| Area | Mainline status |
| --- | --- |
| HTTP/REST requests | Supported |
| GraphQL requests | Supported through HTTP POST body construction |
| SSE and streaming HTTP responses | Supported through streaming MIME detection |
| AI requests | Supported through `genai` and `better_networking` |
| WebSocket requests | Supported |
| Dashbot assistant | Supported |
| cURL, Postman, Insomnia, HAR import | Supported |
| OpenAPI-assisted import | Supported through Dashbot services |
| Code generation | Supported for cURL, HAR, C, C#, Dart, Go, JS, Java, Julia, Kotlin, PHP, Python, Ruby, Rust, Swift |
| Environment Variables | Supported |
| Request History | Supported |

## Repository Map

| Path | Responsibility |
| --- | --- |
| `lib/main.dart` | Startup: Stac init, model discovery, settings load, Hive init, desktop window init, provider overrides |
| `lib/app.dart` | App shell, theme, workspace selector, desktop/mobile routing, close/save behavior |
| `lib/consts.dart` | App constants, feature labels, codegen language enum, import format enum, response view routing |
| `lib/models/` | App-level Freezed models: request, history, settings |
| `lib/providers/` | Riverpod state for requests, environments, history, settings, UI, terminal, JS runtime |
| `lib/services/` | Hive, shared preferences, history retention, window services, agentic helper calls |
| `lib/screens/` | Feature pages and responsive desktop/mobile page composition |
| `lib/widgets/` | Reusable app widgets and response preview widgets |
| `lib/codegen/` | Request-to-code generation router and language implementations |
| `lib/importer/` | Import dialog and facade over cURL/Postman/Insomnia/HAR parsers |
| `lib/dashbot/` | Dashbot UI, chat state, prompts, actions, OpenAPI/cURL apply flows |
| `lib/terminal/` | Terminal/log data models and terminal widgets |
| `packages/` | Workspace packages shared by the app |
| `test/` | Root app unit and widget tests |
| `integration_test/` | Desktop/mobile integration tests |
| `doc/dev_guide/` | Contributor-facing setup, testing, architecture, and extension guides |
| `doc/proposals/` | Proposal material; not implementation source of truth |

## Package Map

API Dash uses a Melos workspace with the root app plus 12 packages.

| Package | Responsibility |
| --- | --- |
| `packages/seed` | Shared low-level models such as `NameValueModel` and `FormDataModel` |
| `packages/better_networking` | HTTP/GraphQL/SSE execution, auth, request/response models, cancellation, OAuth callback server |
| `packages/genai` | Unified AI request model, provider adapters, model manager, agentic engine |
| `packages/apidash_core` | Import/export adapters, environment model, shared core utilities |
| `packages/apidash_design_system` | Design tokens, typography, themes, context extensions, base widgets |
| `packages/curl_parser` | cURL parsing and Dart-to-cURL conversion helpers |
| `packages/postman` | Postman collection models and parsing utilities |
| `packages/insomnia_collection` | Insomnia export models and parsing utilities |
| `packages/har` | HAR 1.2 models and utilities |
| `packages/json_explorer` | Interactive JSON explorer widget |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foss42/apidash](https://github.com/foss42/apidash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
