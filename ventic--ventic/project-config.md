---
trigger: always_on
description: Nuxt 4 + Vuetify 4 front end, Tauri 2 (Rust) back end. A media library over an
---

# Ventic

Nuxt 4 + Vuetify 4 front end, Tauri 2 (Rust) back end. A media library over an
embedded BitTorrent engine, playing into a real mpv window that the Rust side
keeps glued to a box in the page. Targets desktop **and Android TV**.

## Working here

- `bun` only (`preinstall` enforces it). `bun run tauri:dev` for the app,
  `bun run dev` for the browser-only front end, `bun run lint` before finishing.
- `bun run build` makes native bundles, `bun run build:windows` a cross-compiled
  `.exe`, `bun run build:android` an APK. `.msi` and macOS need their own OS.
  There is no CI — builds are local. See the README for what plays where.
- The mpv backend has one file per windowing system: X11 in
  `src-tauri/src/player.rs`, Win32 in `player_windows.rs`, macOS in
  `player_macos.rs` (+ `player_render_mac.rs`), and stubs in
  `player_unsupported.rs` for Android. Keep all four in sync when you add or
  change a `player_*` command, or the other targets stop compiling. The IPC
  socket the two unix backends share is `player_socket.rs`.
- **macOS is not a child window, it is libmpv in-process.** The platform embeds
  no other process's window and mpv's Cocoa output takes no `--wid`, so there is
  no process to parent: the app links libmpv, sets `vo=libmpv`, and renders the
  frames itself into an `NSOpenGLView` (`player_render_mac.rs`). Two rules follow.
  The view goes **under** the WKWebView, whose background is switched off while a
  film is up — so macOS uses Android's compositing (`behind`, the `ventic-video`
  class, DOM input) with mpv's protocol, and punches no cutouts. And AppKit, the
  GL context and `mpv_render_context` are all the **main thread's**: hop with
  `run_on_main_thread`, and never hold the `PlayerState` lock across a hop that
  needs it back.
- Control is still the IPC socket even there — libmpv honours
  `input-ipc-server`, so `player_socket.rs` and every command the frontend sends
  work unchanged. The C API is used only to create the handle, load the file and
  tear it down. `player_status` has no process to watch, so it asks
  `idle-active` instead.
- You can type-check the macOS build from Linux, which is worth doing before
  claiming it works: `rustup target add aarch64-apple-darwin` then
  `cargo check --target aarch64-apple-darwin`. It gets as far as
  `objc2-exception-helper`, which compiles a `.m` file — point
  `CC_aarch64_apple_darwin` at any stub that writes an object file (nothing is
  linked during a check).
- Where mpv can't be run at all (Android, a plain browser) the player falls
  back to the webview's `<video>`. `app/utils/htmlvideo.ts` answers the *same*
  mpv command/property protocol, so `MpvPlayer.vue` is one component with one
  `native` flag rather than two players — a new control needs no second
  implementation, but a new mpv property does need a line in the shim's `READ`.
  `bun run check:player` covers the translation.
- **`MpvPlayer.vue` is one component and four composables**, because four of
  the things it was doing are not about being a player at all.
  `useNativeSurface` is a box on a page, a scale factor and the holes the
  chrome shows through — it owns the *only* copy of that arithmetic, so
  `player_start` placing the window and the frame loop keeping it there can
  never disagree about where the picture is (`check:player` fails on a second
  one). `useSeekPreview` is ffmpeg and a piece bitfield, and answers
  `heldSpan` as well as the hover frames because both want the same cached
  bitfield. `usePlayerAudio` is a per-title setting and one filter chain.
  `useSubtitleSync` is the delay, the rate and what the last pass had to say.
  What is left is the component's actual job: controls, a template, and the
  `native` flag. Each is built where its first reader is rather than under its
  own heading — a `const` destructure is a TDZ throw, not a style note.
- **A decoder may say yes and then die, so Android carries its own.** Android
  answers `format_supported=YES` for E-AC-3 on a device whose only decoder is
  the vendor's Dolby one (`media_codecs_dolby_c2.xml` is the sole file on a
  Pixel that mentions eac3), and that decoder then rejects the first frame of
  streams FFmpeg decodes without a complaint — `work failed to complete: 14`,
  fourteen milliseconds after a clean configure. There is nothing for
  `setEnableDecoderFallback` to reach for, because it only covers a decoder that
  fails to *initialise* and there is no second decoder anyway, and a WEB-DL
  usually carries one audio track — so one frame ended the whole film. That is
  what "the same link plays in other apps" meant: every other Android player
  ships FFmpeg. `retryInSoftware` in `Player.kt` is the answer, and the shape of
  it matters. The device decoder is tried **first** and the FFmpeg renderer only
  after it has actually failed, because `FfmpegAudioRenderer` answers
  FORMAT_HANDLED without ever asking whether the sink could have passed the
  stream through — preferring it from the start would quietly end Dolby
  passthrough for every TV wired to a receiver. The flag resets per film, so one
  bad track costs the next one nothing, and the retry is posted rather than run
  inside the listener that reported the error. The decoder itself is not on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ventic/ventic](https://github.com/ventic/ventic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
