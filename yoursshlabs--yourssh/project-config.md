---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Run (macOS / Windows / Linux)
cd app && flutter run -d macos
cd app && flutter run -d windows
cd app && flutter run -d linux

# Build
cd app && flutter build macos
cd app && flutter build windows
cd app && flutter build linux

# Lint / analyze
cd app && flutter analyze

# Tests
cd app && flutter test
cd app && flutter test test/services/sync_service_test.dart   # single test file
cd app && flutter test --name "pattern"                       # filter by test name
```

## Makefile targets (Rust core — inactive/future)

```bash
make setup          # Install deps (Rust targets, xcodegen)
make core           # Build universal .a + Swift bindings
make swift-bindings # Regenerate Swift bindings only
make open           # Generate Xcode project and open it
make clean          # Remove Rust build artifacts + generated bindings
```

## Architecture

The active codebase is `app/` — a Flutter app targeting macOS, Windows, and Linux. The `core/` Rust library is **not currently used at runtime**; it was built in Sprint 1 and kept for future `flutter_rust_bridge` integration.

**Monorepo layout:**
- `app/` — the Flutter app
- `packages/dartssh2` — **local fork** of dartssh2; overrides the pub.dev version via `dependency_overrides` in `app/pubspec.yaml`
- `packages/yourssh_plugin_api` — abstract plugin interface (`YourSSHPlugin`, `YourSSHPluginContext`)
- `packages/yourssh_devops` — DevOps plugin (containers (Docker/K8s), network tools, Cloudflare tunnel, mail catcher, MCP server, S3 browser)
- `packages/yourssh_web_tools` — Web Tools plugin (in-app browser over port-forwarded HTTP)
- `packages/yourssh_snippets` — Snippets plugin
- `packages/yourssh_script_engine` — JS plugin runtime (QuickJS FFI, HookBus, bridges, PluginLoader, PermissionGuard)

**Data flow:**

```
Flutter UI (widgets/screens)
  └── Providers (ChangeNotifier, via provider package)
        └── SshService / StorageService
              └── dartssh2 (SSH, SFTP, port forwarding) [local fork]
              └── flutter_secure_storage (Keychain / Credential Manager)
              └── shared_preferences (host list, app settings)
```

**Providers** (`app/lib/providers/`):
- `HostProvider` — CRUD for saved SSH hosts; fires `onMutation` callback to trigger sync push
- `SessionProvider` — manages active `SshSession` objects; wires key lookup, auto-reconnect, tmux, and host-key verification via callbacks set in `main.dart`
- `KeyProvider` — SSH key entries (path + optional passphrase + optional linked certificate path)
- `PortForwardProvider` — local/remote/dynamic `PortForward` tunnel configs (persistent rules)
- `TunnelProvider` — active `TunnelConfig` sessions (runtime state, separate from PortForwardProvider)
- `SnippetProvider` — reusable command snippets (managed by `yourssh_snippets` plugin)
- `SyncProvider` — holds Supabase sync config (URL/key, optional passphrase, status); `enabled` is derived from `isSupabaseConfigured` — no separate stored flag; passphrase stored in secure storage via `StorageService`
- `KnownHostsProvider` — persists known host fingerprints; exposes `pendingChallenge` for TOFU dialog
- `SettingsProvider` — app-wide prefs (auto-reconnect, tmux, hotkeys, feature flags for DevOps/WebTools/Snippets)
- `TerminalLayoutProvider` — split layout (none/horizontal/vertical) and input bar visibility
- `LocalSessionProvider` — manages local shell sessions via `flutter_pty`
- `LocalFilePanelProvider` — local filesystem state for the dual-panel SFTP view
- `SftpPanelProvider` — remote SFTP panel state (current path, directory listing)
- `SftpTransferProvider` — in-progress upload/download transfer queue and status
- `CommandHistoryProvider` — per-host command history for terminal autocomplete
- `AiChatProvider` — AI chat sidebar; supports multiple providers (`AiProvider` enum: `anthropic`, `openai`, `gemini`); API keys and model selection stored per-provider in `SharedPreferences`
- `PluginProvider` — activates/deactivates registered plugins; wraps `PluginContextImpl` for each
- `PluginEngineProvider` — wires `ScriptEngineService` into the Flutter state tree; surfaces loaded JS plugins, enabled state, and per-plugin console logs to the UI
- `RecordingProvider` — recording library state; `startRecording(session)` / `stopRecording(sessionId)`; `refreshLibrary()` scans disk for `.cast` files; `isRecording(sessionId)` for UI indicators; wired to `SessionProvider` via `recordingStart` callback in `main.dart`

**Services** (`app/lib/services/`):
- `SshService` — owns `SSHClient` and `SSHSession` maps keyed by host ID; handles connect, shell, exec, sftp, `testConnection` (TCP+auth without opening a shell), disconnect
- `StorageService` — host list as JSON in `SharedPreferences`; all secrets via `_saveSecret/_loadSecret/_deleteSecret` helpers (secure-first: write to `FlutterSecureStorage`, purge stale prefs copy on success, fall back to prefs on error); exposes `saveGenericSecret` / `loadGenericSecret` / `deleteGenericSecret` for app-scoped secrets (e.g., `sync_passphrase`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YoursshLabs/yourssh](https://github.com/YoursshLabs/yourssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
