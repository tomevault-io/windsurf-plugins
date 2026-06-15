---
trigger: always_on
description: - `Package.swift` is the source of truth for app builds and tests.
---

# Ironsmith Agent Guide

## Build And Test

- `Package.swift` is the source of truth for app builds and tests.
- Build and stage the development app with `script/build.sh`.
- Build and run the development app with `script/build.sh run`.
- Build a Developer ID-signed release app with `script/build.sh --release --sign-identity "Developer ID Application: Example (TEAMID)"`.
- Package and notarize the release app with `script/package.sh --sign-identity "Developer ID Application: Example (TEAMID)"` plus one complete notarization credential option group; pass an explicit `.app` path to package another bundle.
- Run tests with `script/test.sh`.
- Remove SwiftPM/script outputs with `script/clean.sh`.
- Build-time backend values are documented in `Config/.env.example`; local `Config/.env` is gitignored.
- App tests use Swift Testing (`@Test`, `#expect`, `#require`) rather than XCTest.

## Project Shape

- Ironsmith is a macOS menu bar app built with SwiftUI, SwiftData, Observation, Foundation Models, AnyLanguageModel, and a small AppKit bridge.
- The menu bar surface is `IronsmithMenuBarController`, an `NSStatusItem` plus `NSPopover`. Do not reintroduce `MenuBarExtra` unless the app shell is deliberately changed.
- `Ironsmith/App` owns app-level wiring. `IronsmithApp` should stay focused on model container setup, startup bootstrapping, shared state creation, the AppKit menu bar controller, and the SwiftUI `Settings` scene.
- `Ironsmith/Core/Models` contains persisted SwiftData model types and stable domain identifiers: `Tool`, `ProviderConfig`, and `ModelConfig`. Do not rename persisted model types or stored properties casually, including `ProviderConfig.baseURLString` with its `originalName`.
- `Ironsmith/Core/Persistence` owns SwiftData container creation, app data bootstrapping, filesystem paths, preference keys, and the small `ToolRepository`.
- `Ironsmith/Core/Inference` owns provider catalog metadata, inference state, repository access, credentials, account/credit state, remote model discovery, local MLX downloads, Ollama integration, model selection, generation preferences, and language model construction.
- `Ironsmith/Core/AgentPipeline` owns generated-tool scaffolding, metadata prompts, source cleanup, Swift package builds, compiler diagnostic parsing, deterministic repairs, optional model-diff repairs, app bundle building, icon generation, launch/export clients, version backups, and diagnostics logging.
- `Ironsmith/Features/Launch` owns launch routing and Xcode Command Line Tools detection/onboarding.
- `Ironsmith/Features/ToolLibrary` owns the compact menu bar popover, `ToolLibraryStore`, tool rows, Finder/export actions, restore-previous-version actions, prompt composition, launch state, and export state.
- `Ironsmith/Features/Settings` owns the settings scene content, provider/model sections, sheets, presentation helpers, and small reusable settings controls.
- Tests mirror this structure under `IronsmithTests/Core` and `IronsmithTests/Features`; add focused tests beside the behavior you change.

## Architecture Pattern

- Keep the app simple: views render state and send intent; stores coordinate workflows; repositories wrap data access and persistence; closure clients wrap effectful service/process/filesystem operations.
- `InferenceStore` is the shared `@Observable` state owner for inference. It exposes providers, persisted local models, transient remote models, selected model state, provider connection issues, Ollama transfer state, Ironsmith account/session/credit state, error/smoke-test state, `GenerationPreferencesStore`, and `ModelSelectionStore`.
- `ToolLibraryStore` is local `@Observable` state for the popover only. Do not put tool-list UI state, selected tool state, generation progress, export state, launch state, restore availability, prompt text, or sandbox toggle state into `InferenceStore`.
- `InferenceRepository` is the normal persisted-data access layer for providers and persisted local models. It currently uses SwiftData, but callers should depend on the repository boundary rather than storage details. It should not make network calls, touch Keychain, launch processes, or persist remote model discovery results.
- `AppDataBootstrapper` is startup seeding logic, not a general repository. It ensures app directories and baseline built-in data exist when their runtime dependencies are available.
- Closure clients are the main side-effect seams:
  - `CredentialClient` for Keychain-backed API keys.
  - `IronsmithAccountClient` for app-side account, session, credit pack, checkout, and account deletion operations.
  - `RemoteModelClient` for provider model-list requests, provider-specific headers, response decoding, and text-model filtering.
  - `LocalModelClient` for MLX HuggingFace Hub filesystem/download work.
  - `OllamaClient` for detecting/starting Ollama and pulling/deleting Ollama models.
  - `LanguageModelClient` for constructing/running AnyLanguageModel language models.
  - `ToolGenerationClient`, `ToolRunnerClient`, `ToolBuildClient`, `ToolExportClient`, `ToolFinderClient`, and `ToolVersionBackupClient` for generated-tool effects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jeidoban/Ironsmith](https://github.com/Jeidoban/Ironsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
