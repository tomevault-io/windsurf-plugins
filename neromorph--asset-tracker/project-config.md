---
trigger: always_on
description: This file governs how the AI agent writes, reviews, and refactors code for this project.
---

# CLAUDE.md — AI Agent Coding Guide
## Asset Tracker · Flutter + Supabase Cloud

This file governs how the AI agent writes, reviews, and refactors code for this project.
Read it fully before generating any code. Follow every rule without exception unless
explicitly overridden by the user in the current session.

---

## 1. Project Identity

| Field | Value |
|---|---|
| App name | Asset Tracker (working title) |
| Purpose | Family asset manager with billing and document expiry tracking |
| Users | Private family use (owner + spouse), ~2 active users |
| Platforms | iOS and Android (Flutter) |
| Backend | Supabase Cloud (free tier, `ap-southeast-1` Singapore region) |
| Timezone | `Asia/Jakarta` (WIB, UTC+7) — all date logic must respect this |
| Language | Dart (Flutter), SQL (Supabase migrations) |
| Min SDK | Flutter 3.19+, Dart 3.3+, iOS 16+, Android API 26+ |

---

## 2. Architecture Principles

### 2.1 Folder Structure

```
lib/
├── main.dart                  # Entry point only — no business logic
├── app.dart                   # MaterialApp, ThemeData, routing setup
├── core/
│   ├── constants/             # App-wide constants (strings, colors, sizes)
│   ├── errors/                # Failure classes, AppException
│   ├── extensions/            # Dart extension methods
│   ├── theme/                 # ThemeData, text styles, color scheme
│   └── utils/                 # Pure utility functions (date, format, etc.)
├── data/
│   ├── datasources/
│   │   ├── local/             # SQLite / Drift local DB
│   │   └── remote/            # Supabase client calls
│   ├── models/                # JSON-serializable data models
│   └── repositories/          # Concrete repository implementations
├── domain/
│   ├── entities/              # Pure domain objects (no JSON, no DB)
│   ├── repositories/          # Abstract repository interfaces
│   └── usecases/              # Single-responsibility use case classes
├── presentation/
│   ├── pages/                 # Full screens
│   ├── widgets/               # Reusable UI components
│   └── providers/             # Riverpod providers / state notifiers
└── services/
    ├── notification_service.dart
    ├── sync_service.dart
    └── auth_service.dart
```

### 2.2 State Management

- Use **Riverpod 2.x** (`flutter_riverpod`, `riverpod_annotation`) exclusively.
- No `setState` outside of purely local, ephemeral UI state (e.g., a text field focus).
- All providers must be typed. Never use `dynamic` or `Object` as provider return types.
- Use `AsyncNotifier` for async operations; `Notifier` for synchronous state.
- Keep providers in `presentation/providers/` colocated with the feature they serve.

### 2.3 Repository Pattern

- Every data operation goes through a repository interface in `domain/repositories/`.
- Concrete implementations live in `data/repositories/`.
- Use cases (`domain/usecases/`) call repositories only — never Supabase directly.
- Presentation layer calls use cases only — never repositories directly.
- This layering is non-negotiable. Do not shortcut it even for simple reads.

### 2.4 Offline-First

- Local SQLite (via **Drift**) is the single source of truth for the UI.
- All reads come from local DB. All writes go to local DB first, then sync to Supabase.
- Sync is eventual — the app must be fully functional with no internet connection.
- Mark records with `sync_status`: `synced | pending_create | pending_update | pending_delete`.
- Sync service runs on app foreground and on connectivity restore.

---

## 3. Security Rules — Non-Negotiable

### 3.1 Secrets and Credentials

- **Never hardcode** Supabase URL, anon key, or any secret in Dart source files.
- Store secrets in `--dart-define` at build time or in a `.env` file loaded via `flutter_dotenv`.
- `.env` must be in `.gitignore`. Commit a `.env.example` with placeholder values only.
- Never log secrets, tokens, or user PII to the console — not even in debug mode.

```dart
// CORRECT
final supabaseUrl = const String.fromEnvironment('SUPABASE_URL');
final supabaseAnonKey = const String.fromEnvironment('SUPABASE_ANON_KEY');

// WRONG — never do this
final supabaseUrl = 'https://xxxx.supabase.co';
```

### 3.2 Supabase Row Level Security (RLS)

- Every Supabase table MUST have RLS enabled. No exceptions.
- Default policy: deny all. Only grant what is explicitly needed.
- Every user can only access rows where `owner_id = auth.uid()` OR where they belong to the same `family_group_id`.
- Write migration SQL for every policy alongside the table creation migration.
- Never disable RLS even temporarily for debugging — use the Supabase dashboard to inspect data instead.

```sql
-- Example RLS policy pattern
alter table assets enable row level security;

create policy "Users can view their own family assets"
  on assets for select
  using (
    owner_id = auth.uid()
    or family_group_id in (
      select family_group_id from users where id = auth.uid()
    )
  );
```

### 3.3 Authentication

- Use Supabase Auth exclusively. No custom auth logic.
- Support email/password and Google OAuth.
- On logout: clear local DB sensitive data, invalidate all Riverpod providers, navigate to login.
- Session tokens are managed by the Supabase Flutter client — do not store them manually.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neromorph/asset-tracker](https://github.com/neromorph/asset-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
