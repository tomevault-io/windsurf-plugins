---
trigger: always_on
description: >
---


# Jetpack Compose & Compose Multiplatform

This skill covers the full Compose app development lifecycle — from architecture and state management through UI, networking, persistence, performance, accessibility, cross-platform sharing, build configuration, and distribution. Jetpack Compose and Compose Multiplatform share the same core APIs and mental model. **Not all Jetpack libraries work in `commonMain`** — many remain Android-only. A subset of AndroidX libraries now publish multiplatform artifacts (e.g., `lifecycle-viewmodel`, `lifecycle-runtime-compose`, `datastore-preferences`), but availability and API surface vary by version. **Before adding any Jetpack/AndroidX dependency to `commonMain`, verify the artifact is published for all required targets by checking Maven Central or the library's official documentation.** CMP uses `expect/actual` or interfaces for platform-specific code. MVI (Model-View-Intent) is the recommended architecture, but the skill adapts to existing project conventions.

## Existing Project Policy

**Do not force migration.** If a project already follows MVI with its own conventions (different base class, different naming, different file layout), respect that. Adapt to the project's existing patterns. The architecture pattern — unidirectional data flow with Event, State, and Effect — is what matters, not a specific base class or framework. Only suggest structural changes when the user asks for them or when the existing code has clear architectural violations (business logic in composables, scattered state mutations, etc.).

## Workflow

When helping with Jetpack Compose or Compose Multiplatform code, follow this process:

1. **Read the existing code first for context** — check conventions, base classes, and layout. For small UI or logic asks, restrict your reading to the immediately relevant files to save time. Do not map out the entire project architecture unless a structural refactor is requested.
2. **Identify the concern** — is this architecture, state modeling, performance, navigation, DI, animation, cross-platform, or testing?
3. **Apply the core rules below** — the decision heuristics and defaults in this file cover most cases.
4. **Consult the right reference** — load the relevant file from `references/` only when deeper guidance is needed. Use the [Quick Routing](#quick-routing) in the Detailed References section to pick the right file.
5. **Verify dependencies before recommending** — before adding or upgrading any dependency, verify coordinates, target support, and API shape via a documentation MCP tool or official docs (see [Dependency Verification Rule](#dependency-verification-rule)).
6. **Flag anti-patterns contextually** — if the user's code violates best practices, call it out for production code. For quick prototypes or minor UI tweaks, prioritize answering their specific question over lecturing them on strict rules.
7. **Write the minimal correct solution** — do not over-engineer. Prefer feature-specific code over generic frameworks.

## Dependency Verification Rule

**Before recommending any new dependency or version upgrade, verify:**

1. **Coordinates** — Confirm the exact Maven coordinates (`group:artifact:version`) exist and are current.
2. **Target support** — Confirm the artifact supports the project's targets (Android, iOS, Desktop, `commonMain`). Do not assume a Jetpack library works in `commonMain` unless verified.
3. **API shape** — Confirm the API you plan to use actually exists in that version. Function signatures, parameter names, and return types change between major versions.

**How to verify:**
- **Documentation MCP tool** (preferred) — If a documentation MCP server is available (e.g., Context7), verify exact tool names and schemas first, then use it to fetch current official documentation for the library.
- **Official docs** — Search the library's official documentation or release notes.
- **Maven Central / Google Maven** — Check artifact availability and supported platforms.

**If verification is not possible** (no documentation tool, no network access, docs unavailable), **provide the standard or latest known dependency snippet anyway.** Add a brief comment (e.g., `// Verify latest version`) so the user isn't blocked.

## Fetching Up-to-Date Documentation

When adding a new dependency, upgrading major versions, or verifying latest API patterns, use a **documentation MCP tool** (e.g., Context7) if available. Before invoking, verify the tool's exact name and parameter schema — tool names vary across environments.

1. **Resolve library ID** — if the tool requires a resolution step, call it first.
2. **Query docs** — call with the resolved ID and a specific question.

**Alternative**: Users can add `use context7` (or equivalent) to their prompt. Bundled references remain the primary source for architectural patterns and MVI guidance; use documentation tools for API-specific and version-specific queries.

## Core Architecture: MVI or MVVM

Both MVI and MVVM use **unidirectional data flow**: UI renders state → user acts → ViewModel updates state → UI re-renders. The difference is how UI actions reach the ViewModel.

- **MVI**: `sealed interface Event` + single `onEvent()` entry point
- **MVVM**: Named public functions (`onTitleChanged()`, `save()`)

Both patterns use:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meet-Miyani/compose-skill](https://github.com/Meet-Miyani/compose-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
