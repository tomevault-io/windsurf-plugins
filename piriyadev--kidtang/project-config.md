---
trigger: always_on
description: > **AI Instructions**: Read this file before making any changes. Follow the existing architecture. Prefer modifying existing patterns over creating new ones. Avoid unnecessary refactoring. Keep changes minimal and consistent. Reuse existing utilities whenever possible. Preserve backward compatibility unless explicitly requested. Search the repository before implementing duplicate functionality. Explain breaking changes before applying them.
---

# AGENTS.md — Kidtang Project Context

> **AI Instructions**: Read this file before making any changes. Follow the existing architecture. Prefer modifying existing patterns over creating new ones. Avoid unnecessary refactoring. Keep changes minimal and consistent. Reuse existing utilities whenever possible. Preserve backward compatibility unless explicitly requested. Search the repository before implementing duplicate functionality. Explain breaking changes before applying them.

---

## 1. Project Overview

- **Name**: Kidtang (กิดตัง) — "มาจ่ายเงินกัน" (Let's split the bill)
- **Purpose**: A bill-splitting app that lets users create bills, add members and line items, calculate each person's share (with VAT, service charge, tip, discount), track payments, and manage groups of friends.
- **Main Features**:
  - Create and manage bills with line items, custom splits, and per-item payer tracking
  - Bill lifecycle: `draft` → `pending_payment` → `completed`
  - Group management (shared bill spaces)
  - Friends system with invite/accept flow
  - In-app notifications (group invites, friend requests, bill events)
  - Firebase push notifications (FCM)
  - Multi-currency support (default THB)
  - Bilingual UI: Thai (default) and English
  - Dark/light theme
  - PromptPay QR code generation for payment
  - Google AdMob integration (remotely toggled via `app_config`)
- **Supported Platforms**: iOS, Android, Web (PWA installable)

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Language | Dart (SDK ≥3.0) |
| Framework | Flutter (Material 3) |
| State Management | `provider` ^6.1.2 — `ChangeNotifier` + `MultiProvider` |
| Navigation | `go_router` ^13.2.0 — `StatefulShellRoute` for bottom nav |
| Backend / DB | Supabase (Postgres + RLS + Realtime + Edge Functions) |
| Auth | Supabase Auth (email/password, Google, LINE, Apple) |
| Push Notifications | Firebase Messaging (FCM) + `flutter_local_notifications` |
| Charts | `fl_chart` ^0.68.0 |
| Fonts | `google_fonts` — Anuphan (headings), NotoSansThai (body) |
| i18n | Manual JSON files in `assets/i18n/` — no codegen |
| Ads | Google Mobile Ads (`google_mobile_ads`) — remotely toggled |
| Build / Deploy | `run.sh` (local), `deploy.sh` + Netlify (web), standard `flutter build` (mobile) |
| Env Config | `.env` (mobile local dev) + `--dart-define` (web/CI) |

---

## 3. Repository Structure

```
lib/
  main.dart              # App entry: init Supabase, Firebase, LINE SDK, providers
  router.dart            # GoRouter config — auth-gated redirects, all routes
  firebase_options.dart  # Generated Firebase config

  models/                # Pure data classes — no Flutter deps
    models.dart          # Barrel export for all models
    bill/                # Bill, BillItem, BillMember, BillSettings, BillCalculation
    group/               # Group, GroupMember
    friend/              # Friend
    me/                  # Profile
    shared/              # AppConfig, AppNotification

  providers/             # ChangeNotifier providers (auth, theme, locale, notifications)
    auth_provider.dart   # Auth state, profile loading, social sign-in delegation
    theme_provider.dart  # Dark/light toggle, persisted via SharedPreferences
    locale_provider.dart # TH/EN locale, synced from profile
    notifications_provider.dart  # In-app notification list + unread count

  stores/                # ChangeNotifier stores — in-memory cache + optimistic updates
    bills_store.dart     # Single source of truth for all bills; Realtime subscriptions
    groups_store.dart    # Groups + group members
    friends_store.dart   # Friends list + pending requests

  repositories/          # Pure Supabase I/O — no state, no Flutter deps
    bills_repository.dart
    groups_repository.dart
    friends_repository.dart

  services/              # Stateless helpers and platform-conditional services
    app_config_service.dart      # Remote feature flags (ads_enabled) from Supabase
    social_auth_service.dart     # Google + LINE sign-in orchestration
    profile_repository.dart      # Profile CRUD (used by AuthProvider)
    push_notification_service.dart  # FCM token save/clear, notification handling
    line_web_auth_service.dart   # LINE PKCE web flow
    line_web_platform.dart       # Conditional import facade (web vs stub)
    google_web_button.dart       # Conditional import facade (web vs stub)
    ios_install_prompt.dart      # Conditional import facade (web vs stub)

  screens/               # Full-page widgets, one per route
    shared/              # login, onboarding, main_shell, line_web_return
    home/                # HomeScreen
    bill/                # BillsScreen, BillDetailScreen, CreateBillScreen
    group/               # GroupsScreen, GroupDetailScreen, CreateGroupScreen
    friend/              # FriendsScreen, NotificationsScreen
    me/                  # MeScreen, ProfileScreen


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PiriyaDEV/kidtang](https://github.com/PiriyaDEV/kidtang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
