---
trigger: always_on
description: Native Android client for the Downtify self-hosted music server. Kotlin, Jetpack Compose + Material 3, Hilt, Media3, Room, DataStore. See README.md for build and setup.
---

# Downtify Android — conventions

Native Android client for the Downtify self-hosted music server. Kotlin, Jetpack Compose + Material 3, Hilt, Media3, Room, DataStore. See README.md for build and setup.

## The server contract

- The contract lives in the server repo: `~/git/downtify/docs/mobile-client-contract.md`, with details in `docs/api-reference.md` ("Server and sign-in", "Mobile API (v1)", "WebSocket") and user-facing behaviour in `docs/features/mobile-apps.md`.
- **Don't invent endpoints.** If the app needs something the server doesn't offer, write it in `docs/server-requirements.md` and work around it (or leave the feature out).
- When the docs and the server code disagree, the code wins; note the mismatch in `docs/server-requirements.md`.
- **Never modify the server repo from here.** Read it only. To test against it, run it with `DOWNLOAD_DIR`/`DATABASE_DIR` pointed at a scratch directory and a spare `--port`; port 8000 may be the user's own instance. From 3.2 a fresh server has accounts: sign in as `admin`/`downtify` (`POST /api/auth/login`, keep the cookie, send an `Origin` header) to create pairing codes with `POST /api/auth/pairing`.
- Every request carries `Authorization: Bearer <token>` (`AuthInterceptor`), even when the server doesn't require sign-in: a 401 is how the app learns it was unpaired. A 401 "revoked", a WebSocket close 4401, or a refused WebSocket handshake confirmed by `GET /api/auth/status` all end the session (`SessionStore.onRevoked`).

## Modules and dependency rules

```
:app ──► :core:designsystem
     ──► :core:player ──► :core:data ──► :core:network ──► :core:model
```

- `:core:model` is pure Kotlin (JVM): no Android imports. Put logic here whenever it can live without Android — it's the cheapest place to test. It runs on Android too, so no JVM APIs newer than API 26 (e.g. `URLEncoder.encode(String, Charset)` is API 33 — use the `"UTF-8"` overload). Android lint does not check this module; be careful.
- `:core:network` knows the wire format (DTOs, Retrofit interfaces, WebSocket, NSD). DTOs never leave it: map to `:core:model` types. Two interfaces: `DowntifyApi` for the mobile client contract (`/api/v1`, pairing, listens, activity) and `WebApi` for the web app's own routes a device may call (search, downloads, queue, podcasts, Discover). `ApiFactory.createWebPatient` is for calls the server answers only when it's done (an episode download, Discover's first run).
- `:core:data` owns persistence and the sync/report logic. Repositories expose `Flow`/`StateFlow`; the UI never touches DAOs or the API directly.
- `:core:player` owns Media3. The UI talks to playback only through `PlayerController`.
- Screens live in `:app` under `feature/<name>/` (Screen + ViewModel). There are no feature modules yet: at this size they'd add build config without buying isolation. Split a feature out when it gains its own data layer (e.g. offline downloads).
- Gradle config goes in the convention plugins in `build-logic/`; versions only in `gradle/libs.versions.toml`. Check current stable versions before bumping — don't guess.

## Architecture

- Single activity, Navigation Compose with type-safe `@Serializable` routes (`ui/navigation/Routes.kt`).
- MVVM + unidirectional data flow: a ViewModel exposes one `StateFlow<XUiState>`; the screen calls ViewModel functions for events.
- Split every screen into a `XRoute` (gets the ViewModel, collects state) and a stateless `XScreen(state, callbacks…, modifier)` that previews and tests can call directly.
- **Text fields:** keep the text in synchronous snapshot state (`var x by mutableStateOf("")` in the ViewModel, or a plain `MutableStateFlow`), never in a flow that went through `combine`/`stateIn` — the async hop drops keystrokes. Don't rewrite the typed text (e.g. uppercase) in `onValueChange`; use a `VisualTransformation`.
- Coroutines: `viewModelScope` in ViewModels, the `@ApplicationScope` scope for app-lifetime work. Rethrow `CancellationException` from every broad `catch`.
- Android 17+ (target 37): talking to anything on the LAN needs the runtime `ACCESS_LOCAL_NETWORK` permission. The connect screen asks for it before starting NSD discovery.
- Server search and requests: `CatalogRepository` and `ServerQueueRepository` (`:core:data/catalog`). The web API's song objects aren't versioned like `/api/v1`: read them leniently (`CatalogJson`) and send them back to the server untouched (`RemoteSong.raw`), never rebuilt from the app's fields. The queue is read-only for devices.
- Podcasts (`PodcastsRepository`) and Discover (`DiscoverRepository`) go through `callServer`/`ServerResult` like the catalog. A device can only read podcasts, download an episode and save progress — subscribing is admin-only, so don't offer it. An episode is a media item with id `episode:{id}` (`EpisodeIds`) playing the server's file; anything that assumes a media id is a library track (likes, lyrics, listens, the notification's heart) must check `EpisodeIds.isEpisode` first. Progress is saved by `EpisodeProgressTracker` in the service, not the UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henriquesebastiao/downtify-android](https://github.com/henriquesebastiao/downtify-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
