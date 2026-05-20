---
trigger: always_on
description: This file tells an AI coding agent how to be immediately productive in the EpubReader repo (a cross-platform .NET MAUI EPUB reader). Keep edits focused, preserve style, and follow project conventions.
---

# Copilot Instructions

## EpubReader — Copilot instructions (concise)

This file tells an AI coding agent how to be immediately productive in the EpubReader repo (a cross-platform .NET MAUI EPUB reader). Keep edits focused, preserve style, and follow project conventions.

- **Big picture**: .NET MAUI app with MVVM. Core responsibilities are split into:
  - UI & platform shims: `Platforms/`, `Views/`, and `Controls/` (platform files use suffixes like `.android.cs`, `.macios.cs`).
  - Business logic & parsing: `Service/EbookService.cs` (VersOne.Epub + SixLabors.ImageSharp integrations).
  - Persistence & sync: `Database/Db.cs`, `Service/FirebaseSyncService.cs`, and secrets in `build-secrets/`.
  - Messaging & state: `ViewModels/` (MVVM Toolkit attributes) and `Messages/` (WeakReferenceMessenger).

- **Key files to inspect for most tasks**:
  - `Service/EbookService.cs` — EPUB parsing, cover extraction
  - `Database/Db.cs` — SQLite initialization and models
  - `MauiProgram.cs` — DI registration and platform wiring
  - `ViewModels/` — look for `[ObservableProperty]` and `[RelayCommand]`
  - `Messages/` — message classes used with `WeakReferenceMessenger`

- **Build / Run**
  - Primary build script: `build.ps1`. Example debug build: `pwsh -File ./build.ps1 -ApiKey <key> -AuthDomain <domain> -DatabaseUrl <url> -Configuration Debug` - VS Code tasks available: "Build with Firebase Secrets", "Build (Release) with Firebase Secrets", "Build from .env file", and "Run app (Windows)".
  - Secrets live in `build-secrets/google-services.json` or are passed via `build.ps1` flags. Do not commit secrets.

- **Repository conventions (must follow)**
  - File-scoped namespaces (e.g., `namespace EpubReader.Service;`).
  - Fields use camelCase, no underscore prefix.
  - Public async methods accept `CancellationToken token = default` and call `token.ThrowIfCancellationRequested()` early.
  - Use `Trace.WriteLine()` for logging (not `Debug.WriteLine()`).
  - Enums: index 0 should be `Unknown`/`Default` for safe deserialization.

- **MVVM / messaging patterns**
  - ViewModels inherit `BaseViewModel : ObservableObject`.
  - Properties use `[ObservableProperty]` and commands use `[RelayCommand]` (source-generated code used broadly).
  - Prefer `[ObservableProperty]` for bindable view model properties in this repo.
  - Cross-VM communication uses `WeakReferenceMessenger.Default.Send(...)` with message classes under `Messages/`.
  - ViewModels often `Dispose()` and unregister from messages.
  - Use view models and MVVM patterns for UI components in this repo.

- **Platform integration notes**
  - Platform-specific behavior is implemented via files with platform suffixes (`*.android.cs`, `*.macios.cs`, `*.windows.cs`).
  - `MauiProgram.cs` performs DI registration and may call `FirebaseConfigLoader.InjectFirebaseSecrets()` under `#if ANDROID`.

- **When modifying code**
  - Make focused, minimal edits. Preserve style and public APIs.
  - Prefer using repository source-generator patterns (`[ObservableProperty]`, `[RelayCommand]`) over hand-rolled implementations.
  - Run `build.ps1` (or the VS Code tasks) after changes that affect the build.

- **Helpful examples to copy from**
  - `ViewModels/BookViewModel` — shows `[ObservableProperty]`, `[RelayCommand]`, and messenger usage.
  - `Service/FirebaseSyncService.cs` — demonstrates offline queueing and reconciliation.

- **What to avoid**
  - Introducing `NotImplementedException` in production code.
  - Committing any Firebase or secret files into source control.

If anything is unclear or you want more examples/patches, tell me which area to expand and I will iterate.

## Overview
Guidelines for AI agents contributing to **EpubReader**, a cross-platform .NET MAUI EPUB reader with cloud sync, Calibre server integration, and multi-platform support (Windows, Android, iOS, macOS).

## Architecture Overview

### Core Layers
- **EbookService** (`Service/EbookService.cs`): Handles EPUB parsing via VersOne.Epub library, cover extraction, font/image embedding, and synthetic page numbering.
- **Database** (`Database/Db.cs`): SQLite wrapper for book metadata, settings, and sync state; auto-initializes tables on first use.
- **Authentication** (`Service/AuthenticationService.*.cs`): Platform-specific implementations for Google Firebase auth; supports local-only mode without authentication. Note: `Plugin.Firebase.Auth.Google` 3.1.2 is incompatible with `Plugin.Firebase.Auth` 5.x. For Google sign-in with `Plugin.Firebase.Auth` 5.x, implement providers directly with the native platform SDK and pass native credentials into `Plugin.Firebase.Auth`.
- **Sync** (`Service/FirebaseSyncService.cs`): Manages reading progress sync across devices; queues offline changes, reconciles on reconnect. When cloud progress has a newer timestamp and a different reading position, the app should always ask to switch to it on book open, without requiring a different device check.
- **MVVM**: ViewModels inherit `ObservableObject` (MVVM Toolkit); communicate via `WeakReferenceMessenger` using message classes in `Messages/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ne0rrmatrix/EpubReader](https://github.com/ne0rrmatrix/EpubReader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
