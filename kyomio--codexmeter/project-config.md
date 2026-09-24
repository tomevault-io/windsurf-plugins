---
trigger: always_on
description: This repository is the Android project for **CodexMeter** (`codexbar-apk`): a self-use / small-scale sideloaded Android 12+ app for monitoring AI provider quota and balance usage across multiple providers (started Codex-only; Codex remains the primary provider).
---

# CodexMeter Agent Guide

This repository is the Android project for **CodexMeter** (`codexbar-apk`): a self-use / small-scale sideloaded Android 12+ app for monitoring AI provider quota and balance usage across multiple providers (started Codex-only; Codex remains the primary provider).

This file is the canonical operating guide for AI agents and human maintainers working in this repo. `CLAUDE.md` must stay aligned with this file.

## 1. Mandatory reading order

Before making any non-trivial change, read these files in order:

1. `docs/PRD.md` — product scope and non-goals.
2. `docs/ARCHITECTURE.md` — architecture, data, auth, refresh and security boundaries.
3. `docs/SPEC.md` — implementation contract, work order, acceptance and verification.
4. `docs/CODEX_DEVICE_CODE_LOGIN_SPEC.md` — required when touching Codex login, OAuth session, account connection, auth UI, auth notifications or legacy `auth.json` migration.
5. `RULES.md` — development quality, maintainability, testing and safety rules.
6. `DESIGN.md` — required for any UI / Widget / Notification / UX work.
7. `AGENTS.md` — this guide.

If these documents conflict, stop and update the documents or ask for a decision. Do not guess.

## 2. Product baseline

- Display name: `CodexMeter`.
- Project directory: `codexbar-apk`.
- Package name: `com.kmnexus.codexmeter`.
- Debug package name: `com.kmnexus.codexmeter.debug`.
- Target platform: Android 12+.
- Distribution: self-use / small-scale sideloaded APK.
- MVP provider: Started Codex-only; now ships 10 providers (Codex, DeepSeek, z.ai Coding Plan, z.ai API, MiniMax, Cursor, Kimi, Claude, Antigravity, Grok) registered in ProviderRegistry.

MVP core surfaces:

- App dashboard.
- Resizable home-screen Widget.
- Low-noise persistent notification.

## 3. Hard product boundaries

Do not implement these unless the PRD is explicitly revised:

- No public app-store distribution.
- No Android 12- support.
- No tablet / foldable / Wear OS support in MVP.
- No lock-screen quota surface.
- No web-page parsing.
- No token / cost local estimation.
- No Cookie Header manual input.
- No access-token / refresh-token manual input fields.
- No embedded WebView / manual-credential Codex login flow; Codex verification uses the provider device-code external-browser handoff.
- No cloud sync, export report, third-party proxy, remote logging or analytics.
- No real-time multi-account monitoring in MVP; periodic background refresh may refresh all Active accounts with bounded concurrency.

Allowed quota data sources only:

- Codex / OpenAI OAuth session obtained through the Hermes-aligned device-code external-browser login flow.
- Existing saved OAuth sessions, including sessions originally imported from Codex CLI `auth.json` before this migration; keep them refreshable and do not auto-delete them.
- Official usage / quota API data returned by the provider.

## 4. Architecture baseline

Use the architecture in `docs/ARCHITECTURE.md`.

Technology baseline:

- Kotlin.
- Jetpack Compose + Material 3.
- StateFlow + ViewModel.
- Room for business history.
- DataStore for preferences.
- Android Keystore + AES-GCM for sensitive session payloads.
- OkHttp + kotlinx.serialization for network and DTO parsing.
- WorkManager for background refresh.
- Jetpack Glance for Widget.
- Hand-written `AppContainer` for dependency assembly.

MVP must not introduce:

- Hilt.
- Retrofit.
- Ktor Client.
- Foreground Service.
- Dynamic plugin system.
- Multi-module Gradle architecture.
- Remote analytics / crash upload / ad SDK.

## 5. Package boundaries

Follow the package map from `docs/ARCHITECTURE.md`:

```text
com.kmnexus.codexmeter
├── app
├── core
├── data
├── domain
├── notification
├── providers
├── refresh
├── ui
└── widget
```

Rules:

- `ui` must not directly access Room, DataStore, OkHttp, Keystore or provider-private sessions.
- `widget` must not directly access network, providers or decrypted sessions.
- WorkManager glue in `refresh` must go through `RefreshCoordinator` and must not directly create notifications.
- Provider-private token, DTO and endpoint details must stay under `providers.<providerId>`.
- Common UI / Widget / Notification state must derive from `CurrentQuotaState` or its clipped read models.

## 6. Security red lines

Never commit, log, print, show in UI, place in screenshots, write into fixtures or include in diagnostics:

- access token.
- refresh token.
- id token.
- OAuth authorization code.
- Cookie.
- complete `auth.json`.
- complete OAuth callback query string.
- raw usage API response body.

Rules:

- Legacy raw `auth.json` content must never be persisted and remains relevant only for redaction / diagnostics handling.
- Do not add `auth.json` import UI, file picker, paste-JSON flow or manual credential path.
- Do not add single-token input fields.
- Do not add Cookie Header input fields.
- Diagnostics must be redacted and safe to copy.
- Release builds must not print request / response bodies.

## 7. UI / UX baseline

Follow `DESIGN.md`.

Confirmed design direction:

- Air Glass Dashboard.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KyoMio/CodexMeter](https://github.com/KyoMio/CodexMeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
