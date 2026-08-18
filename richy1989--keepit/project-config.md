---
trigger: always_on
description: A notes app: a React web frontend and a native Android client over a shared ASP.NET Core REST API,
---

# keepIT

A notes app: a React web frontend and a native Android client over a shared ASP.NET Core REST API,
with real-time sync across a user's devices. Notes can be **shared between users** (owner + Viewer/Editor
grants), and support checklists, colors, pins/archive/trash, per-user lists, reminders, and search.
**Read `ARCHITECTURE.md` before any structural work** — it holds the design and the reasoning; this
file is only the always-on rules.

Three deployables live in this repo: the **API** (`keepIT/keepITCore/`), the **web app** (`web/`),
and the **Android app** (`app/`). They talk only over HTTP + WebSocket — never share code or a process.

## Stack

- **Backend:** ASP.NET Core Web API on **.NET 10** — `keepIT/keepITCore/` (solution `keepIT/keepITCore.slnx`).
- **Data:** EF Core → **PostgreSQL** in prod, **SQLite** dev fallback (provider chosen at startup from config). All backend-written data (SQLite file, media, Data Protection keys) lives under `App__DataRoot`.
- **Auth:** ASP.NET Core Identity + JWT. Access token in the response body, held in memory; refresh token in an httpOnly cookie; silent refresh on 401.
- **Realtime:** SignalR `RealTimeHub` at `/api/realtime` (JWT via `?access_token=`, per-user delivery). After a mutation the API pushes `Changed(resources)`; clients invalidate the matching cache keys and refetch.
- **Web frontend:** React 19 + Vite + TypeScript — `web/`. TanStack Query owns all server state. Tailwind v4, React Router v7.
- **Android client:** Kotlin + Jetpack Compose (Material 3) — `app/` (package `org.hyperstarit.keepitapp`). Retrofit + OkHttp + kotlinx.serialization for the REST API, the official Microsoft SignalR Java client for realtime, Glance for the home-screen widget, Navigation Compose for nav. **Offline-first** with a local cache + mutation outbox. No Room, no Hilt — see the Android section.
- **API contract:** OpenAPI from C# → clients. **C# DTOs are the single source of truth.** Web regenerates a typed client with `openapi-typescript`; the Android `data/Dtos.kt` is hand-kept in sync with the same DTOs.
- **Deploy:** Docker Compose — nginx (`web`) serves the SPA and reverse-proxies `/api` to the API; Traefik in front for TLS. Also shipped as a single self-contained image for Unraid (`deploy/keepit.unraid.xml`).

## Hard rules (backend + web)

- **Never hand-write TypeScript that mirrors C# DTOs.** Change a DTO → `npm run generate:api` → fix the TypeScript errors (they are the complete list of call sites). Drift is a bug.
- **Server data lives in TanStack Query** (web) — never a global store (no Redux/Zustand/context for fetched data).
- **Frontend and backend are separate deployables** over HTTP + WebSocket. Never host React inside ASP.NET Core.
- **Note edits are optimistic** — instant UI, rollback on error.
- **Refresh token stays in the httpOnly cookie.** Never put any token in localStorage.
- **Note access is "own OR shared", never a bare `OwnerId == me`.** Resolve every note endpoint's access through `NoteAccessService` (`Notes/NoteAccessService.cs`): read needs ownership or any share; content writes need ownership or an **Editor** share; hard-delete is owner-only. Pin/archive/trash and list membership are **per-user** — write the caller's `NoteUserState` / `NoteList` row, not the shared note.
- **A new mutating endpoint must push realtime.** After `SaveChangesAsync`, call `IRealtimeNotifier.NotifyAsync(userId, …)` with the affected resources (`notes` / `lists` / `notification`). For a **shared** note's content, fan out to the whole recipient set (`NoteAccessService.RecipientIdsAsync`, i.e. owner + grantees); for **per-user** changes notify only the caller — mirror the existing controllers, or devices won't resync.

## Android app (`app/`)

Mirror the web app's behavior; it's a peer client, not a port. When the web app gains a note capability,
the Android app generally should too. Key design points:

- **Offline-first.** The whole dataset lives in memory as `StateFlow<List<NoteDto>>` in `NotesRepository`, backed by `data/offline/`: `LocalStore` persists a JSON `CacheSnapshot` + the outbox to `filesDir/offline/` (atomic temp-file+rename, **deliberately not Room** — personal-note scale). Mutations enqueue a `PendingOp` in the `Outbox`; `SyncEngine` replays them when connectivity returns (`ConnectivityMonitor`) or on foreground/ sign-in. Go through the repository — never call the API directly from UI.
- **DTOs are hand-synced.** `data/Dtos.kt` mirrors the C# DTOs (the source of truth). Change a C# DTO → update `Dtos.kt` to match. There is no codegen step here, so this is the one place drift can creep in — keep field names and nullability exactly aligned.
- **Session** (`SessionRepository` + `ApiClient`): access token in memory, refresh cookie persisted in app-private `SharedPreferences` via `PersistentCookieJar` (the mobile analogue of the web httpOnly cookie), silent refresh on 401. Base server URL is user-entered at login.
- **Realtime** (`RealtimeClient`): SignalR against `RealTimeHub`; on `Changed` it triggers a sync/refetch, same contract as the web client.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Richy1989/keepIT](https://github.com/Richy1989/keepIT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
