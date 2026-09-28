---
trigger: always_on
description: Agent instructions and project context. This is the single source of truth for how to work on this repository; `CLAUDE.md` and other tool-specific files point here.
---

# AGENTS.md — Lumina

Agent instructions and project context. This is the single source of truth for how to work on this repository; `CLAUDE.md` and other tool-specific files point here.

## 1. Project Overview

- **Name**: Lumina — a lightweight EPUB reader.
- **Platforms**: Android and iOS.
- **Framework**: Flutter (Dart SDK `^3.10.8`, Flutter `>=3.44.6`).
- **Native core**: Rust, bridged with `flutter_rust_bridge` (§5).

### Language requirement — CRITICAL

**All generated code, inline comments, doc comments, identifiers, commit messages and UI strings must be written in English.** The two exceptions are the localized payload files (`lib/l10n/app_localizations_zh.dart` and the `.arb` sources) and the architecture documents under `docs/AGENTS/`, which are written in Chinese.

## 2. Repository Layout

```text
lib/
  main.dart                     app entry: RustLib.init, AppStorage.init, Isar warm-up, WebView pre-warm
  l10n/                         generated localizations (en, zh)
  src/
    app.dart                    MaterialApp.router root
    router.dart                 GoRouter route table (app assembly — see layering rules)
    providers.dart              Isar schema list + keychain store (app assembly — see layering rules)
    core/                       shared infrastructure — must not know about any feature
      database/                 Isar lifecycle (schemas injected at the assembly root)
      platform/                 native file picking, PlatformPath, import cache
      providers/                app-wide providers
      services/                 ToastService, UrlLauncher, StorageCleanupService
      storage/                  AppStorage paths, AppStorageConstants
      theme/                    AppTheme, color schemes, theme notifier
      widgets/                  cross-feature widgets
    features/
      library/                  bookshelf, import pipeline, shelf data owner
        domain/ data/ application/ presentation/
      backup/                   library backup export, restore, storage cleanup
        data/ application/ presentation/
      fonts/                    imported custom fonts (import, list, delete)
        domain/ application/ presentation/
      external_sources/         remote book sources (WebDAV today)
        domain/ data/ application/ presentation/
      reader/                   WebView reading engine
      detail/                   book detail screen
      settings/                 settings UI only (no domain of its own)
        presentation/
    rust/                       flutter_rust_bridge generated Dart bindings
    web/                        generated web-asset bundle + WebView bridge
      api/
rust/                           Rust crate `lumina_rust`
rust_builder/                   Flutter FFI plugin shell around cargokit
web_assets/                     TypeScript/CSS sources inlined into lib/src/web/web_assets.dart
tool/                           repository scripts
docs/AGENTS/                    architecture deep-dives (see §9)
```

### Layering rules

- **`core/` must not import from `features/`.** This holds with **no exemptions**. When a `core/` service needs feature data, inject it as a callback or through a provider that lives in the feature. Code that inherently has to name features — the route table, the app root — belongs at the `src/` root (`app.dart`, `router.dart`) or inside a feature, not in `core/`.
- **`lib/src/` root is the assembly layer.** `main.dart`, `app.dart`, `router.dart` and `providers.dart` wire the app together and are allowed to import features. Keep it to those; anything reusable belongs in `core/`, anything domain-specific in a feature.
- Within a feature, dependencies flow `presentation → application → data → domain`. `domain/` depends on nothing but Isar annotations.
- **No dependency cycles between features.** The intended shape is acyclic and it currently is: `backup → library`, `detail → library`, `external_sources → library` (import pipeline only), `fonts → library`, `library → external_sources` (import menu entries only), `reader → library`, `reader → fonts`, `settings → {backup, external_sources, fonts, library}`. If you need an edge that would close a cycle, move the shared piece down into `core/`, move the *caller* into the feature that already owns the dependency, or — when two features genuinely have to name each other, as the schema list does — hoist that one piece to the `src/` assembly root.
- Cross-feature reuse goes through `core/` only for genuinely generic capability. Domain models are not "generic": `ShelfBook` lives in `features/library/domain/`.
- Feature modules use a four-layer split: `domain/` (Isar entities and pure models), `data/` (repositories, services, parsers), `application/` (notifiers and business logic), `presentation/` (screens, widgets, mixins). A feature creates only the layers it actually needs — `settings/` has just `presentation/`.
- A feature may expose a composable section widget (for example `BackupSection`, `FontsSection`) so a host screen can embed it without reaching into the feature's internals.

## 3. Reading Engine (Red Line)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MilkFeng/lumina](https://github.com/MilkFeng/lumina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
