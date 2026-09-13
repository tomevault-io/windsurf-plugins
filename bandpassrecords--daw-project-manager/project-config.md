---
trigger: always_on
description: Instructions for AI assistants working on this codebase.
---

# CLAUDE.md — DAW Project Manager

Instructions for AI assistants working on this codebase.

---

## Non-negotiables

### Tests are part of every task
- Every bug fix must include a regression test that would have caught the bug.
- Every new feature must include tests covering its logic in full.
- Run `flutter test` before every commit — only commit if all tests pass.
- If logic is too deep in the widget tree to unit test, write the closest possible test (model/service/repository level) and note explicitly what cannot be automated.

### Always use AppLocalizations for UI strings
- Never hardcode user-visible text in widget trees.
- Any label, tooltip, snackbar message, dialog text, or button label must have a key in `lib/l10n/app_en.arb` and be referenced via `AppLocalizations.of(context)!.someKey`.
- Add the key to all locale ARB files, then run `flutter gen-l10n`.
- The app ships in 9 languages — hardcoded strings silently break all non-English users.
- `app_pt.arb` is Brazilian Portuguese only — there is a single Portuguese locale for this app, and it targets Brazil. Never introduce European Portuguese spelling or vocabulary (e.g. "ficheiro", "deteção"/"detetar", "descarregar", "ecrã", "utilizador", "está a fazer"-style progressive) — use the Brazilian equivalent instead (e.g. "arquivo", "detecção"/"detectar", "baixar", "tela", "usuário", "está fazendo"). When adding or editing a `pt` string, match the vocabulary already established in the rest of `app_pt.arb`.

### New model fields must be evaluated for Drive sync AND local backup
- When adding a field to `MusicProject` (or any synced model), decide: is this user-generated data or a device-local preference?
- **User data** (metadata, timers, todos, paths) → add to `_serializeProject` AND `_deserializeProject` in `lib/services/google_drive_sync_service.dart`. If skipped, the field is silently lost on every Drive restore.
- **Device-local settings** (which theme is selected, layout, update checks) → do NOT sync. `AppSettings` fields are generally device-local.
- The split can run through one feature: a **custom theme's definition** is user data (synced and backed up), while **which theme is selected** is a device-local preference (neither). Ask the question per field, not per feature.
- Also check `lib/services/backup_service.dart` (local file export/import) for the same question, for any *global* (non-per-profile) data — `TodoTemplate`, `ProjectTemplate`, `TemplateRoot`, custom mixdown folder names, custom themes, phase settings. This is Flatpak's only backup path (see below), so a field skipped here is a field Flatpak users can never back up at all, not just "won't survive a Drive restore."
- Merging is **union, never deletion**: something present locally but absent in the incoming data is something made since, not something removed. On a same-id collision the newer `updatedAt` wins. `mergeCustomThemes` is the reference implementation.

### Google Drive sync is not offered inside Flatpak
- `GoogleDriveSyncService.isSupported` is `false` only when actually running inside a Flatpak sandbox (detected via the `/.flatpak-info` marker file every Flatpak app has at runtime), `true` everywhere else — including the plain Linux tarball and the AppImage. Every UI entry point to `GoogleDriveSyncPage` (dashboard, profile page, startup dialog, tray menu) is gated on it — don't add a new one without the same gate.
- Why: the desktop OAuth flow requires a client secret (confirmed against a live Google sign-in attempt — PKCE alone isn't accepted for this app's "Desktop app" OAuth client type; see the long comment on `_desktopClientSecret` in `google_drive_sync_service.dart`). Flathub's build sandbox has no secret-injection mechanism, so shipping the secret there would mean committing it in the open in the public Flathub submission repo. Not offering the feature was chosen over that for Flatpak specifically — the tarball/AppImage are built by this repo's own CI (`build_linux` in `release.yml`), same trust boundary as Windows/macOS, so they get the real secret too.
- The Flatpak build compiles `lib/config/oauth_config.dart` from the committed template's placeholder text directly (no real values, no GitHub secrets involved) — see `flatpak/README.md`. The Linux tarball/AppImage build (`build_linux`) injects real values from GitHub secrets like every other desktop platform.

---

## Architecture

### State management — Riverpod
- All providers live in `lib/providers/providers.dart`.
- Main data flow: `repositoryProvider` → `allProjectsStreamProvider` → `projectsProvider` (filtered/sorted) → UI.
- Search state is per-tab: `projectsSearchProvider`, `releasesSearchProvider`, `queueSearchProvider`, etc.

### Local persistence — Hive CE
- Models with `@HiveType` / `@HiveField` and a `part` directive (e.g. `Release`, `TodoTemplate`) use code-generated adapters — run `dart run build_runner build --delete-conflicting-outputs` after changing them.
- `MusicProject` and other models have manually written `TypeAdapter`s inline in their model file — `build_runner` skips these intentionally.
- Adapters are registered in `ProjectRepository` with `isAdapterRegistered` guards to prevent double-registration.

### Platform detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bandpassrecords/daw-project-manager](https://github.com/bandpassrecords/daw-project-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
