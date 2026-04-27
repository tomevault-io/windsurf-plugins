---
trigger: always_on
description: The Swift package lives in `src/`. Run all `swift build`, `swift run`, and `swift package` commands from there, not from the repo root.
---

# AGENTS.md

## Build & Run

The Swift package lives in `src/`. Run all `swift build`, `swift run`, and `swift package` commands from there, not from the repo root.

```bash
# Debug build
cd src && swift build

# Run the app (menu bar app — swift run does not work for LSUIElement apps)
# Build the .app bundle first, then open it:
./create-app-bundle.sh && open DroidProxy.app

# Release .app bundle at repo root
# Picks up CODESIGN_IDENTITY / APP_VERSION / TARGET_ARCH from env when present
./create-app-bundle.sh
```

`create-app-bundle.sh` currently builds `DroidProxy.app` at the repo root and bundles resources from `src/Sources/Resources/`.

### Notarization (local)

```bash
ditto -c -k --sequesterRsrc --keepParent "DroidProxy.app" "DroidProxy-notarize.zip"
xcrun notarytool submit "DroidProxy-notarize.zip" --keychain-profile "notarytool" --wait
xcrun stapler staple "DroidProxy.app"
```

### Sparkle update signing

```bash
src/.build/artifacts/sparkle/Sparkle/bin/sign_update DroidProxy-arm64.zip
```

## Source Of Truth

The compiled app code is under `src/`. Treat `src/Sources/**`, `src/Info.plist`, and `create-app-bundle.sh` as source of truth.

There is also a top-level `resources/` tree containing mirrored Swift files and assets. It is not the SwiftPM target used by `src/Package.swift`, so do not assume edits there affect the app unless the task is explicitly about that directory.

## Architecture

DroidProxy is a macOS menu bar app (`LSUIElement`) with:

1. `ThinkingProxy` on `localhost:8317`, the user-facing TCP proxy.
2. Bundled `CLIProxyAPIPlus` on `127.0.0.1:8318`, managed as a child process by `ServerManager`.

Typical request flow:

`Client -> :8317 ThinkingProxy -> :8318 CLIProxyAPIPlus -> upstream provider`

### Current ThinkingProxy behavior

`ThinkingProxy.swift` no longer implements the old `-thinking-N` suffix parser described in older docs.

What it does today:

- Inspects `POST` JSON requests for supported Claude, Codex GPT, and Gemini models
- Injects Claude adaptive thinking for models whose name contains `opus-4-7` or `sonnet-4-6`
- Injects `"thinking":{"type":"adaptive"}` and forces `"stream":true` for Claude
- Injects `"output_config":{"effort":"..."}`
- Reads effort from `AppPreferences.opus47ThinkingEffort` or `AppPreferences.sonnet46ThinkingEffort`
- Injects Codex reasoning for exact models `gpt-5.3-codex` and `gpt-5.4`
- Injects `"reasoning":{"effort":"..."}`
- Reads effort from `AppPreferences.gpt53CodexReasoningEffort` or `AppPreferences.gpt54ReasoningEffort`
- Injects Gemini thinking levels for `gemini-3.1-pro-preview` and `gemini-3-flash-preview`
- Rewrites the model name to append a suffix (e.g. `gemini-3.1-pro-preview(high)`) which CLIProxyAPIPlus parses via its `ParseSuffix` logic
- Reads level from `AppPreferences.gemini31ProThinkingLevel` or `AppPreferences.gemini3FlashThinkingLevel`
- Optionally injects `"service_tier":"priority"` for `gpt-5.4` on Responses API paths (`/v1/responses`, `/api/v1/responses`) when `AppPreferences.gpt54FastMode` is enabled and the client did not already set `service_tier`
- Preserves JSON key order by editing the raw JSON string instead of re-serializing
- **Max Budget Mode**: When `AppPreferences.claudeMaxBudgetMode` is enabled, forces streaming and applies a Sonnet-4.6-only override: classic extended thinking with `budget_tokens=63999` / `max_tokens=64000` / `effort=max`. Opus 4.7 is unaffected and continues to receive `thinking.type=adaptive` with `output_config.effort` from `AppPreferences.opus47ThinkingEffort`.

What it does not do anymore:

- It does not strip or normalize model suffixes
- It does not send `thinking.budget_tokens` to Opus 4.7 (rejected with 400)
- It does not add `anthropic-beta` interleaved-thinking headers (adaptive thinking enables interleaving automatically)
- It does not implement the old `-thinking-N` / suffix-based branching documented in stale docs

### Amp routing

`ThinkingProxy` also handles Amp-specific routing:

- `/auth/cli-login` and `/api/auth/cli-login` are redirected directly to `https://ampcode.com/...`
- `/provider/*` is rewritten to `/api/provider/*`
- Requests that are not provider requests and not `/v1/*` or `/api/v1/*` are treated as Amp management requests and forwarded to `ampcode.com`
- Amp response `Location` headers and cookie domains are rewritten so browser flows continue working through localhost

## Auth And Providers

The current app/UI exposes three provider types:

- `claude`
- `codex`
- `gemini`

Auth data lives in `~/.cli-proxy-api/` as JSON files. `AuthManager` scans that directory and reads fields like:

- `type`
- `email`
- `login`
- `expired`
- `disabled`

Behavior to know:

- Multiple accounts per provider are supported
- Per-account disable/enable is supported via the `disabled` field in each auth JSON
- The last enabled account for a provider cannot be disabled
- Provider-level toggles in `SettingsView` are separate from per-account disable flags
- Provider-level disable writes `oauth-excluded-models` into `~/.cli-proxy-api/merged-config.yaml`
- `CLIProxyAPIPlus` hot-reloads config changes, so provider enable/disable does not require a restart

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anand-92/droidproxy](https://github.com/anand-92/droidproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
