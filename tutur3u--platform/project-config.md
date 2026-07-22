---
trigger: always_on
description: This file contains Flutter/Dart and mobile-app-specific rules for `apps/mobile`.
---

# AGENTS.md - Mobile Flutter/Dart Playbook

This file contains Flutter/Dart and mobile-app-specific rules for `apps/mobile`.
The authoritative repo-wide instructions live in [`../../AGENTS.md`](../../AGENTS.md) and apply in full here. This file is additive only; if anything here appears to conflict with the root file, follow the root `AGENTS.md`.

## 0. Inheritance

- **Read Order**: Read [`../../AGENTS.md`](../../AGENTS.md) first, then this file.
- **Scope**: Use this file only for `apps/mobile`-specific guidance on top of the root rules.

## 1. Mandatory Actions

- **Localization Generation**: Run `flutter gen-l10n` after ARB changes.
- **Clean Pass**: `bun check:mobile` is mandatory for mobile changes. It runs format, analyze, and test.
- **Analyze Info = CI Failure**: Treat `flutter analyze` info-level lints as blocking in this workspace; fix them in touched files before finishing.
- **Flutter L10n Asset Formatting**: Do not send `.arb` files through `dart format`; format only Dart sources and regenerate `flutter gen-l10n` after ARB edits.
- **Flutter Analyze Ignore Hygiene**: Avoid speculative `ignore_for_file` directives. `flutter analyze` reports `unnecessary_ignore` as a failing issue during `bun check:mobile`.
- **Retrospective - Import Order Lints Can Still Block Delivery**: For new large mobile screens, expect `flutter analyze` style infos such as `directives_ordering`, deprecated form-field APIs, and constructor parameter-order lints to fail `bun check:mobile`. Do not stop at “no errors”; keep touched files info-clean.
- **Retrospective - Shell Chrome Registrations Need Fresh Callback Tokens**: `ShellMiniNavItemSpec` and `ShellActionSpec` equality tracks ids, labels, selected state, and `callbackToken`, but not the callback closure itself. When shell-owned actions or mini-nav items depend on changing page state, update `callbackToken` alongside the visual state or the shell can keep stale handlers registered.
- **Retrospective - Apps Hub Tests Should Assert Ordering, Not Temporary Chrome**: When the mobile Apps hub layout changes, prefer widget tests that verify stable module ordering and the absence/presence of search rather than hero copy, summary banners, or CTA button counts that are likely to churn with visual redesigns.
- **Retrospective - Opportunistic Product Warmups Must Fail Soft**: App-start and Apps hub prewarm tasks are best-effort cache fills, not required product loads. Gate them on the corresponding access cubit when a module is optional, and swallow/log `404` responses from missing endpoints instead of letting those errors escape through startup.
- **Retrospective - Mobile Bearer Routes Must Pass Request Into Workspace Helpers**: Web API routes that serve mobile workspace screens cannot resolve workspaces or permissions through cookie-only helpers. For mobile-facing workspace CRUD/list routes, normalize the workspace id and call `getPermissions({ wsId, request })` / `createClient(request)` so Bearer-token requests do not degrade into false `404 Workspace not found` errors.
- **Retrospective - Mobile Collection Endpoints Must Return Lists, Not `.single()` Payloads**: When porting a web module to Flutter, audit every collection route used by the repository layer. Mobile pagination/search screens expect list-shaped `{ data, count, page, pageSize }` responses; a lingering `.single()` collection handler can look fine in web-only code paths but breaks mobile parsing and paging immediately.
- **Retrospective - Join-Table Mobile Counts Must Use Real Keys On An Admin-Backed Read**: For mobile list APIs that expose derived counts from bridge tables like `course_module_quiz_sets`, do not assume the join table has an `id` column and do not rely on caller-scoped joins after access is already proven. Verify membership with the request client, then read the join on an admin-backed client and select a real foreign key such as `module_id`.
- **Retrospective - Cross-Surface Prompt Cards Should Reuse The Apps Palette Language**: When assistant or onboarding surfaces present suggested actions as tappable cards, prefer the same `AppCardPalette` gradients, icon blocks, and contrast model used in the Apps hub. Reusing that visual language keeps empty states feeling like first-class product entry points instead of neutral placeholders.
- **Retrospective - Home Dashboard Cards Should Share The App-Surface Visual System**: For mobile Home surfaces that route users into modules like Tasks or Calendar, prefer the same `AppCardPalette`-driven gradients, icon containers, and CTA chips used by the Apps hub instead of separate bespoke card styles. Reusing one card language keeps Home, Apps, and Assistant visually coherent.
- **Retrospective - Mobile Task Due Dates Must Persist At Day End**: Task date pickers are date-only, but stored task deadlines are exact timestamps. Normalize mobile task start dates to local start-of-day and due/end dates to local end-of-day before sending them to the API so "due today" does not become overdue immediately after midnight.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tutur3u/platform](https://github.com/tutur3u/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
