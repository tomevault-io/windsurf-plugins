---
trigger: always_on
description: - Keep repository documentation, code comments, identifiers, and commit messages in English. Do not translate existing documentation. The one exception is `README.zh-CN.md`, the Simplified Chinese counterpart of `README.md` (see README below).
---

# AGENTS.md

## Language

- Keep repository documentation, code comments, identifiers, and commit messages in English. Do not translate existing documentation. The one exception is `README.zh-CN.md`, the Simplified Chinese counterpart of `README.md` (see README below).
- iOS and macOS user-facing interfaces support English and Simplified Chinese. On launch, follow the system/app preferred-language list, select the first supported English or Chinese preference, and fall back to English if none matches. Chinese locale variants use Simplified Chinese. Do not hard-code an English UI or persist an independent language override by default.
- Keep app-authored labels, accessibility text, empty states, confirmations, and errors in the shared localization catalog (`shared/locales/zh-CN.ts`) and use `shared/i18n.ts`. Native macOS menus and dialogs use the matching `macos/*.lproj/Localizable.strings` resources. Dates and times use the selected language's locale.
- Text the app writes for the model on the user's behalf, such as the prompt an idea puts in the composer, follows the selected UI language like any other app-authored copy.
- Chinese text is allowed in localization resources and localization tests. Keep protocol names, API fields, resource IDs, file names, and machine-readable values unchanged. Never translate user-authored content, chat history, model output, or raw upstream diagnostic payloads as UI copy.
- Add matching translations and tests when introducing user-facing copy. Verify both languages and the system-language fallback in affected Apple clients.
- When talking to the user, match the language they use in the conversation.

## Project

Open Muse is a personal AI task assistant built on a Managed Agents (MA) service. It ships a mobile-first web app, an iOS app, a macOS native shell, and a retained Android project.

Volcano Ark MA is the default and reference backend; Claude Managed Agents is an alternative selected with `VITE_MUSE_MA_PROVIDER` / `MA_PROVIDER`. Every backend difference lives in `shared/ma-provider.ts`. Ark comes first: build and verify features on Ark, and where Ark has a capability Claude lacks, leave it out on Claude instead of emulating it.

Current focus: the iOS app comes first, the macOS app second. Do not modify the web app or the Android project unless explicitly requested.

Goals for the iOS app:

1. Read iOS health data (HealthKit), including on-demand reads triggered from the conversation, e.g. asking "how was my workout today?" triggers a fresh health query.
2. Carry out arbitrary Lark (Feishu) operations through lark-cli.

Goals for the macOS app:

1. Operate the Mac through computer-use integration.
2. Carry out arbitrary Lark (Feishu) operations through lark-cli.

- `src/` — React UI and direct MA client; `src/direct/` owns local auth, storage, and provisioning
- `shared/` — event types, approval policy, and the MA API catalog/contract
- `server/` — Open Muse service (Cloudflare Worker + D1): Open Muse account verification, per-account encrypted Ark keys, background work
- `ios/` — Capacitor + SwiftPM iOS project
- `macos/` — AppKit/WKWebView shell loading bundled static assets, with no server or Node runtime
- `android/` — retained Capacitor project (no build/device verification yet)
- `tests/` — unit, API, and frontend tests (Vitest)
- `scripts/` — build and asset generation
- `docs/` — integration notes and verification records

Open Muse needs an Open Muse account; there is no single-user or device-key mode. Every app build is configured with `VITE_MUSE_BACKGROUND_URL`, `VITE_MUSE_SUPABASE_URL`, and `VITE_MUSE_SUPABASE_ANON_KEY` (the build scripts read them from the environment or with `ve`), and the account (Supabase Auth email/password) is the user's identity. A build without them cannot connect and says so. The Ark API key is only the model-service credential: it is stored encrypted per account by the Open Muse service, read back only by that account's verified sessions, and scoped with the account owner so accounts sharing one key keep separate workspaces, memory, history, and local records. Clients still call public Volcano Ark APIs directly with that key. The service creates each account's agent, environment, and memory store; clients never create them. An API key or SSO session that an earlier release saved on a device is left untouched and never used. Volcano SSO is not supported. Without credentials the app stays disconnected and never generates simulated replies. Real calls may incur cloud costs. Mock responses and the old server migration harness belong only in tests and must never be bundled.

## README

- `README.md` (English, the default) and `README.zh-CN.md` (Simplified Chinese) introduce the project to new readers: what Open Muse does, why it is worth trying, screenshots, the supported apps, a short getting-started, and links to the docs. Each links to the other on its first lines.
- Keep both READMEs in step: same sections, same claims, same screenshots. Change one, change the other in the same commit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chyroc/open-muse](https://github.com/chyroc/open-muse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
