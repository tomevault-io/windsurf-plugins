---
trigger: always_on
description: This file gives Claude Code the context, constraints, and conventions it must follow when working in this repository. Read it fully before touching any code. Re-read it when starting a new session.
---

# CLAUDE.md

This file gives Claude Code the context, constraints, and conventions it must follow when working in this repository. Read it fully before touching any code. Re-read it when starting a new session.

---

## 1. What this project is

**FreeTube iOS** — a native SwiftUI YouTube client that replicates the YouTube mobile app: home feed, search, playback, login, subscriptions, library, history, playlists, comments, likes, downloads. Plus a "Link" tab for downloading from any of ~2,000 sites supported by yt-dlp.

**Distribution:** TestFlight internal, sideload, or personal use. **Not for public App Store submission.** Do not suggest changes that assume App Store distribution.

**Platforms:** iOS 17.0+, Swift 5.9+, Xcode 15+.

**Bundle identifier:** `com.leshko.freetube`. This is also the reverse-DNS namespace for the `os.Logger` subsystem, the Keychain cookie key (`com.leshko.freetube.cookies`), the `UserDefaults` keys (subscriptions / downloads / metadata), and internal `Notification.Name`s. Keep all of them on this same domain. Changing the identifier invalidates provisioning profiles and resets Keychain / `UserDefaults` state for any prior install.

---

## 2. Non-negotiable constraints

These come first. Violating them breaks the project.

1. **No Google Cloud YouTube Data API.** All YouTube interaction goes through `b5i/YouTubeKit` (cookie-based, no API key). Never suggest `GoogleAPIClientForREST`, `YTPlayerView`, IFrame embeds, or `https://www.googleapis.com/youtube/v3/...`.
2. **No `dimitris-c/AudioStreaming`.** It is for raw audio streams (Icecast/Shoutcast). YouTube serves HLS/DASH. Playback uses `AVPlayer` only.
3. **No App Store assumptions.** Do not add capabilities, entitlements, or workarounds aimed at App Store review (e.g. avoiding `WKWebView` cookie reads, hiding download functionality). Sideload-honest behavior is expected.
4. **No telemetry, analytics, or remote logging by default.** Logs go to `os.Logger` only. Never add SDKs that beacon out.
5. **Cookies are sensitive.** Always store in Keychain via `KeychainHelper`. Never write them to disk, `UserDefaults`, plist, or logs. Never print cookie values.
6. **Signed stream URLs are sensitive and time-limited.** Never persist them. In-memory cache only (`StreamURLCache`), 30-minute TTL max.
7. **MVVM with service layer is mandatory.** Views do not import `YouTubeKit` or `YoutubeDL`. ViewModels do not perform networking directly — they call services in `Core/Networking/`.
8. **SwiftUI only for UI.** No UIKit `UIViewController` subclasses except where bridging is unavoidable (`AVPlayerViewController`, `WKWebView`, `AVPictureInPictureController`). Wrap those in `UIViewControllerRepresentable` / `UIViewRepresentable`.
9. **Swift Concurrency is the default.** Use `async`/`await` and `AsyncSequence`. Use Combine only inside `PlayerStateManager` for `AVPlayer` time observation. Do not introduce RxSwift.
10. **No force unwraps in production code.** Use `guard let`, `if let`, or proper `throw`. Force unwraps allowed only in test fixtures.
11. **`@Observable` is the default for view models, not `ObservableObject`.** Injected through SwiftUI `@Environment(...)`.

---

## 3. Tech stack (locked)

| Layer | Choice |
|---|---|
| UI | SwiftUI (`@Observable`, iOS 17 APIs) |
| Async | Swift Concurrency, `AsyncSequence` |
| Networking (YouTube) | `b5i/YouTubeKit` |
| Stream extraction / download | `kewlbear/YoutubeDL-iOS` (yt-dlp via `PythonKit`) |
| In-process ffmpeg | `FFmpegSupport` (Swift wrapper around ffmpeg C library) |
| Playback | `AVFoundation` / `AVKit` (`AVQueuePlayer`, `AVPlayerViewController`, `AVPictureInPictureController`) |
| Mini / expanded player | `LNPopupUI` |
| Now-playing indicator | `SwimplyPlayIndicator` |
| Login web view | `WebKit` (`WKWebView`, `WKHTTPCookieStore` against ephemeral `.nonPersistent()` data store) |
| Images | `Kingfisher` |
| Secure storage | `Security` (raw Keychain via `KeychainHelper`) |
| Persistence | `SwiftData` (`@Model`); `UserDefaults` for simple flags via `UserPreferences` |
| Background work | `BackgroundTasks` framework + `URLSession` background config (`BackgroundDownloadCoordinator`) |
| Logging | `os.Logger` with subsystem `com.leshko.freetube` |
| JavaScript runtime | `JavaScriptCore` (`JSContext`) — solves YouTube's N/SIG cipher challenges in-process by faking the `deno` runtime to yt-dlp; see §15.11 |

Adding any other dependency requires an explicit ask in the PR description with justification.

---

## 4. Project structure

```
FreeTube/
├── App/
│   ├── AppEnvironment.swift
│   └── RootView.swift
├── FreeTubeApp.swift
├── Core/
│   ├── Networking/        # YouTubeKit wrappers, one Service per response family
│   ├── Auth/              # CookieStore, KeychainHelper, LoginCoordinator,
│   │                      # SessionManager, SubscriptionRegistry, AuthState
│   ├── Player/            # PlayerStateManager, PlaybackResolver, QueueManager,
│   │                      # AudioSessionConfigurator, NowPlayingCenter,
│   │                      # RemoteCommandCenter, StreamURLCache, HLSResourceLoaderDelegate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leshkodev/freetube](https://github.com/leshkodev/freetube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
