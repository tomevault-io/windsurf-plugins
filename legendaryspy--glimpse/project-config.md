---
trigger: always_on
description: Glimpse is a macOS-first Tauri app with a Rust backend and a React/TypeScript
---

Glimpse is a macOS-first Tauri app with a Rust backend and a React/TypeScript
frontend. It is a three-window desktop app, not a routed SPA.

## What matters

- Speed: keep invoke -> record -> transcribe -> insert low-latency.
- Native macOS behavior: preserve menu bar/accessory behavior, shortcuts,
  permissions, focus, and overlay behavior.
- Local-first: transcripts, audio, and API keys stay local by default.
- Simplicity: extend existing owners instead of adding layers.

## Mental model

- Rust owns business logic, native windows, hotkeys, audio, transcription,
  storage, updater, permissions, tray/menu, and privacy-sensitive code.
- React owns rendering, local interaction state, query cache, and thin
  command/event clients.
- The shipping product is macOS-first today. Windows files exist, but they are
  scaffolding.
- Cloud/account paths are mostly future-facing. `TranscriptionMode::Cloud`
  currently normalizes back to local behavior.
- Tooling is intentionally simple: Bun + Vite on the frontend, Cargo + Tauri on
  the backend. Do not add build layers casually.

## Runtime surfaces

- `main`: pill overlay window.
- `toast`: transient toast window.
- `settings`: main settings/history/library window.

Window labels and behavior must stay aligned across:

- `src-tauri/tauri.conf.json`
- `src-tauri/capabilities/*.json`
- `src-tauri/src/lib.rs`
- `src-tauri/src/platform/**/*`
- `src/app/App.tsx`

## Backend ownership

- `src-tauri/src/lib.rs`: composition root, `AppState`, plugin setup, command
  registration, startup loops. Keep it as wiring.
- `src-tauri/src/pill.rs`: shortcut-driven recording lifecycle, overlay state,
  selected-text capture, permission warnings, media pause/resume.
- `src-tauri/src/recorder.rs`: recorder thread, audio preprocessing, validation
  inputs, WAV persistence helpers.
- `src-tauri/src/transcribe.rs`: transcription orchestration, chunking/dedupe,
  completion/error events, storage writes.
- `src-tauri/src/local_transcription.rs`: loaded ASR engine lifecycle. Do not
  duplicate warm/load/unload logic elsewhere.
- `src-tauri/src/model_manager.rs` and `src-tauri/src/downloader.rs`: model
  catalog, artifact layout, install state, downloads.
- `src-tauri/src/assistive.rs`: text insertion and selected-text access.
- `src-tauri/src/mode_context.rs` and `src-tauri/src/accessibility_context.rs`:
  active app/site context for edit and personalization behavior.
- `src-tauri/src/llm_cleanup.rs`: optional second-stage LLM cleanup/edit,
  provider routing, preflight cache.
- `src-tauri/src/settings.rs`: settings schema and persistence in `settings.db`.
- `src-tauri/src/core/settings.rs`: settings validation and post-save side
  effects.
- `src-tauri/src/storage.rs`: transcription/history storage and DB migrations in
  `transcriptions.db`.
- `src-tauri/src/library/repo.rs`: library SQL boundary.
- `src-tauri/src/library/processing.rs`: library filesystem/import/export and
  transcode logic.
- `src-tauri/src/library/queue.rs`: single-flight library queue, progress,
  cancellation, crash-recovery semantics.
- `src-tauri/src/library/commands.rs`: Tauri boundary for library actions.
- `src-tauri/src/dictionary.rs`: dictionary and replacements domain logic.
- `src-tauri/src/personalization.rs`: personalities plus app/site icon
  discovery.
- `src-tauri/src/toast.rs`: toast payloads and toast window positioning.
- `src-tauri/src/tray.rs` and `src-tauri/src/platform/macos/menu.rs`: tray/app
  menu ownership and settings-window accessory behavior.
- `src-tauri/src/update_checker.rs`: background checks, idle gating,
  download/install flow, restart marker.
- `src-tauri/src/crypto.rs`: API-key encryption/decryption.

## Frontend ownership

- `src/app/App.tsx`: routes by Tauri window label, not URL.
- `src/Home.tsx`: settings-window shell and sidebar navigation.
- `src/features/settings/useSettingsForm.ts`: owner of editable settings modal
  state and autosave.
- `src/features/onboarding/machine.ts`: onboarding step flow.
- `src/features/onboarding/OnboardingScreen.tsx`: onboarding side effects and
  initial settings write.
- `src/features/pill/PillOverlay.tsx` and `src/features/pill/machine.ts`: pill
  rendering and frontend state machine.
- `src/features/toast/ToastOverlay.tsx`: toast rendering and actions.
- `src/features/transcriptions/queries.ts` and components: transcription
  history UI.
- `src/features/dictionary/components/DictionaryView.tsx`: current dictionary
  and replacements owner.
- `src/features/personalization/components/PersonalizationView.tsx`: current
  personalization owner.
- `src/features/library/queries.ts`,
  `src/features/library/components/LibraryView.tsx`, and
  `src/features/library/components/LibraryModal.tsx`: main library frontend
  owners.
- Frontend patterns are intentionally mixed by feature:
  - `library` and `transcriptions` are React Query + event-driven
  - `dictionary` and `personalization` are mostly local state + direct invoke
  - `pill` is XState + canvas
  - `toast` is an event-driven window
- `src/shared/lib/*`: static metadata/formatting only, not a service layer.
- `src/shared/ui/*`: small reusable UI primitives.
- `src/types/*`: effective shared frontend type layer today.

## Change map


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LegendarySpy/Glimpse](https://github.com/LegendarySpy/Glimpse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
