---
trigger: always_on
description: This file provides repository-specific guidance for coding agents working in
---

# AGENTS.md

This file provides repository-specific guidance for coding agents working in
this repository.

## Project Overview

- Bangumi-iOS is a SwiftUI iOS app using Swift 6, GRDB, and Xcode.
- The main app target is `Bangumi` in `Bangumi.xcodeproj`.
- `BBCode/` is a local Swift package used by the app for BBCode rendering.
- The app is distributed as `Bangumi Riff` in App Store Connect.

## Workflow Rules

- Communicate with the user in Simplified Chinese by default.
- Keep code, comments, identifiers, commit messages, PR titles, and Markdown code blocks in English.
- Do not run `make format` in this repository unless the user explicitly asks for it. If formatting is needed, keep it local to the touched code.
- Prefer small, reviewable changes that match existing SwiftUI and GRDB patterns.
- Do not include incidental `Bangumi.xcodeproj/project.pbxproj` changes in a PR unless the user asked for a version/build bump or the project file change is truly required.
- Do not use unsafe language features, unsafe concurrency bypasses, or unsafe runtime assumptions anywhere in this repository. This is a hard global requirement. In Swift, this includes `unsafe` APIs, `nonisolated(unsafe)`, `MainActor.assumeIsolated`, unchecked actor isolation workarounds, and similar constructs.
- In non-SQL code, do not use unconstrained interpolated literals inside `map` or `compactMap` followed by `joined()`. GRDB may infer the element type as `SQL` and leak `SQL(elements: ...)` descriptions into rendered output. Extract interpolated fragments into a helper that explicitly returns `String`, or append them to a typed `String` accumulator.

## Storage And Migration Rules

- Persistent storage is backed by GRDB. Do not reintroduce SwiftData storage unless the user explicitly asks for it.
- SwiftData references should stay limited to the one-time legacy import path, such as `LegacySwiftDataMigrator`, old schema definitions, and the old SwiftData migration plan.
- Keep schema setup and migrations centralized in `DatabaseFactory` with `DatabaseMigrator`.
- Once a migration has shipped, treat its registered name and body as append-only history. Do not rename, remove, reorder, or rewrite old migrations.
- For every persistent table, column, index, constraint, or stored payload format change, add a new descriptive `registerMigration(...)` after existing migrations.
- New non-null columns must either have a safe default or be introduced as nullable and backfilled before enforcing non-null behavior.
- For SQLite changes that cannot be expressed safely with `ALTER TABLE`, create a replacement table, copy data explicitly, recreate indexes and foreign keys, then drop/rename inside the migration.
- Treat BLOB or JSON payload shape changes as schema changes. Prefer backward-compatible decoders; otherwise migrate the payloads explicitly or clear only cache tables that are safe to rebuild from the network.
- Validate storage changes with both a fresh database path and an upgraded existing database path, then run `make build`.

### GRDB Migration Discipline

Runtime GRDB migrations in `DatabaseFactory` are historical artifacts and must be treated as immutable once committed.

- Do not mutate already-registered GRDB migrations such as `createGRDBSchemaV1` or `createLocalMigrationMarkers` to add new columns, defaults, indexes, or table shape changes.
- Do not treat helper methods called from baseline migrations, such as `createSubjects` or `createSimpleCaches`, as current-schema builders. They define the frozen baseline for that migration.
- Any GRDB table shape change must be added as a new numbered migration after the latest registered migration. Fresh installs should reach the latest schema by running the baseline migration plus every later migration in order.
- When adding a new persisted field to a GRDB record, update the runtime record model and `CodingKeys` if present, then add a new migration that backfills a safe default for existing databases.
- Validate both upgrade and fresh install paths: an existing `Bangumi.sqlite` must migrate forward, and a new empty database must run all migrations without duplicate-column or missing-column failures.

## Client Architecture Rules

- Keep `APIClient` as the low-level HTTP/session/token client only. Do not add feature-specific business APIs, database loading, Spotlight indexing, UI state, or app runtime state to it.
- Put remote API operations in focused `*Service` types. Put operations that combine remote API calls with local GRDB writes, cache updates, or indexing in focused `*Repository` types.
- Views and components should call services, repositories, `AuthService`, `AppContext`, or other narrow app services. They should not call `APIClient.shared` directly.
- Keep `UserDefaults` access centralized in `AppConfig` using typed static properties. Avoid new raw `UserDefaults.standard` calls outside that boundary; `@AppStorage` remains acceptable in SwiftUI views when it is the local view binding mechanism.
- App runtime state such as the GRDB `DatabaseOperator` and app version display should live behind `AppContext` / `AppMetadata`, not in `APIClient`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bangumi/Bangumi-iOS](https://github.com/bangumi/Bangumi-iOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
