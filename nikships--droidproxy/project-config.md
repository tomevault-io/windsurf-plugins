---
trigger: always_on
description: The Swift package lives in `src/`. Run all `swift build`, `swift run`, and `swift package` commands from there, not from the repo root.
---

# AGENTS.md

## Build & Run

The Swift package lives in `src/`. Run all `swift build`, `swift run`, and `swift package` commands from there, not from the repo root.

```bash
# Preferred dev loop: kill any running DroidProxy, rebuild the .app bundle, and
# launch the freshly signed build. Use this instead of running create-app-bundle.sh
# + open by hand — it guarantees the old menu-bar process and bundled
# cli-proxy-api are stopped before the new app starts.
./dev-relaunch.sh

# Debug build (no .app bundle, no relaunch)
cd src && swift build

# Run the app manually (menu bar app — swift run does not work for LSUIElement apps)
# Build the .app bundle first, then open it:
./create-app-bundle.sh && open DroidProxy.app

# Release .app bundle at repo root
# Picks up CODESIGN_IDENTITY / APP_VERSION / TARGET_ARCH from env when present
./create-app-bundle.sh
```

`dev-relaunch.sh` is the preferred way to run DroidProxy during development. It calls `create-app-bundle.sh` (which runs `swift build -c release` and assembles the signed `.app`) after killing any running `CLIProxyMenuBar` / `cli-proxy-api` processes, then launches the fresh bundle. Do not use it for releases — those go through `.github/workflows/release.yml`.

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

The compiled app code is under `src/`. Treat `src/Sources/**`, `src/Info.plist`, and `create-app-bundle.sh` as source of truth. There is no longer a mirrored top-level `resources/` tree — older AGENTS notes about it are stale.

## Architecture

DroidProxy is a macOS menu bar app (`LSUIElement`) with:

1. `ThinkingProxy` on `localhost:8317`, the user-facing TCP proxy.
2. Bundled `CLIProxyAPI` on `127.0.0.1:8318`, managed as a child process by `ServerManager`.

Typical request flow:

`Client -> :8317 ThinkingProxy -> :8318 CLIProxyAPI -> upstream provider`

### Current ThinkingProxy behavior

Reasoning effort is owned by **Droid CLI**, not the proxy. Each Factory custom model is registered with native reasoning metadata (`enableThinking`, `supportedReasoningEfforts`, `defaultReasoningEffort`, `reasoningEffort`) so Droid's per-session selector exposes every level the model supports, and Droid sends the chosen value in the request body. The proxy does **not** inject `thinking`, `reasoning`, `reasoning_effort`, `output_config`, `budget_tokens`, or `generationConfig.thinkingConfig` for any model — it forwards the request unchanged.

What it still does today:

- **Anthropic-Beta rewriting**: When a Claude request has `thinking.type` of `enabled`/`adaptive`/`auto`, the proxy strips `redact-thinking-2026-02-12` from the `Anthropic-Beta` header and appends the visible-thinking beta list (interleaved-thinking, prompt-caching-scope, fast-mode, etc.). Without this, Claude emits only signed empty thinking blocks.
- **Service tier (fast mode)** for Responses API paths (`/v1/responses`, `/api/v1/responses`): injects `"service_tier":"priority"` for `gpt-5.4` or `gpt-5.5` when `AppPreferences.gpt54FastMode` or `AppPreferences.gpt55FastMode` is enabled and the client did not already set `service_tier`. Fast mode is API priority and is independent of reasoning effort.
- **Gemini path rewrite**: `/v1/responses` (and `/api/v1/responses`) are rewritten to `/v1/chat/completions` for OAuth Code Assist Gemini models (the `-preview`-suffixed names) since CLIProxyAPI does not support those via the Responses API endpoint.
- **Per-request reasoning log** to `/tmp/droidproxy-debug.log`: each `POST` emits a `REQUEST REASONING:` line that extracts just `reasoning` / `reasoning_effort` / `thinking` / `output_config` / `service_tier` / `generationConfig` from the parsed body so the actual values Droid is sending are visible without dumping the whole prompt. Example: `REQUEST REASONING: model=gpt-5.5 reasoning={"effort":"xhigh","summary":"auto"}`.
- Preserves JSON key order by editing the raw JSON string instead of re-serializing (critical for Anthropic's prompt cache). The remaining helpers (`injectJSONField`, `findTopLevelFieldLocation`, etc.) exist for `processOpenAIFastMode`.

What it no longer does (removed in the Droid-CLI-thinking refactor):

- No Claude adaptive thinking injection (Opus 4.8 / Sonnet 4.6 — `thinking` + `output_config`)
- No classic `thinking.budget_tokens` injection
- No Codex `reasoning.effort` injection
- No Gemini `generationConfig.thinkingConfig` injection
- No Kimi `reasoning_effort` injection
- No `claude-opus-4-8(high)` / `gpt-5.2(xhigh)` etc. “advanced variant” suffix parsing — every level now ships in the single base entry via Droid CLI metadata
- No Max Budget Mode override

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikships/droidproxy](https://github.com/nikships/droidproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
