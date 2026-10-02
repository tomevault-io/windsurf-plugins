---
trigger: always_on
description: Windows and macOS video player: **Tauri 2 + Svelte 5 (SvelteKit static) + in-process libmpv** via `tauri-plugin-libmpv` (wid embedding: mpv renders into a child HWND / NSView *behind* the transparent webview; the entire UI is HTML on top). On macOS that needs a patched libmpv — stock mpv does not implement `--wid` there at all; see `patches/` and `scripts/build-macos-libs.sh`.
---

# CLAUDE.md — Frame Player

Windows and macOS video player: **Tauri 2 + Svelte 5 (SvelteKit static) + in-process libmpv** via `tauri-plugin-libmpv` (wid embedding: mpv renders into a child HWND / NSView *behind* the transparent webview; the entire UI is HTML on top). On macOS that needs a patched libmpv — stock mpv does not implement `--wid` there at all; see `patches/` and `scripts/build-macos-libs.sh`.

The UI is **localised (ru/en)** — every user-facing string goes through `t()`, see "Localisation" below. Code comments and docs are **English**.

## Where the documentation is

This file is the index and the short form of the rule book: how the code works
now, and which rules must not be broken, one line each. The **whole** of a rule —
its identifiers, its measurements and the failure it was written against — lives
in **`docs/rules/`**, and every line below that has a rule chapter links to it.
The reasoning *behind* the rules — dead ends, options tried and rejected — lives
in **`docs/`**. All three are part of the repository.

Read the chapter before touching the area it covers; the one-liner here is
enough to know a rule exists, never enough to change the code under it.

| `docs/rules/…` | The rules for |
|---|---|
| `mpv-playback.md` | Driving libmpv: state mirrors, the seek contract, the playlist and the end of a file, chapters, picture geometry, the A–B loop, audio |
| `thumbnails.md` | The seekbar storyboard and every frame decoded outside playback: the budget, the colour space, which frame a hover must show |
| `tracks-and-subtitles.md` | Choosing a track and remembering the choice, subtitle search and placement, closed captions, content languages |
| `torrents-core.md` | The session, adding a torrent, where the data lives, what may be deleted — and the macOS descriptor budget under it |
| `torrents-network.md` | The swarm: DHT, trackers, encryption, the proxy, port forwarding, and the preferences that rebuild the session |
| `torrents-playback.md` | Playing one: the queue, the buffer map, the readouts, subtitles inside a release, replacing a torrent with its re-upload |
| `casting.md` | Google Cast and DLNA: the ladders, the LAN server, the remote-control rules, what one television taught us |
| `sync.md` | Watching together: the wire, the reconciler, readiness, what a room may know |
| `catalog.md` | The metadata proxy, the indexer, and what leaves this machine when somebody searches |
| `sources-and-privacy.md` | What a source *is* against how it is reached, files arriving from the system, the watch history and the seven privacy enforcement points |
| `window.md` | The transparent window, the macOS title bar, fullscreen, the mini player, geometry, the idle and cursor rules |
| `ui-surfaces.md` | The settings sheet, the context menu, the start screen, tooltips, the OSD, and the arithmetic that places anything floating |
| `css.md` | The box model, the cascade between borrowed classes, and the rendering traps measured in both engines |
| `frontend.md` | Where state lives and which way it may depend, the split, hotkeys, localisation, what is worth a test |
| `build-and-release.md` | What ships inside the app, the LGPL/GPL obligations, signing on both platforms, the release workflow |

| `docs/…` | What it holds |
|---|---|
| `architecture.md` | Stack decisions, the embedding model and the constraint list — mandatory reading before touching mpv interop, seeking, fullscreen or zoom |
| `ROADMAP.md` | Shipped, planned and considered features; a comment saying "ROADMAP 21" means a numbered item there |
| `macos.md` | Why stock `--wid` cannot work on macOS, what the patch does, the CoreAudio crash the second patch is a backport for (and the one that was measured and not shipped), and the platform's own traps |
| `sdr-color.md` | SDR against QuickTime, measured: the grey-scale and real-frame numbers, why metadata is not the cause, and which option combinations did and did not reproduce it |
| `casting.md` | Casting to a television: the two transports, what each can carry, the measured limits and the decision rules |
| `watch-together.md` | A shared timeline over a small relay: why the wire carries state rather than actions, why drift is corrected with speed rather than a seek, and what a room may know about what you are watching |
| `catalog.md` | The catalog: why the TMDB key is in a service and not in the player, what the poster traffic costs and who decides where it comes from, how the indexer's fuzzy search is filtered, and the measurements that say a translation-type filter cannot be built |
| `torrents.md` | Torrent streaming: piece priority from playback, what a partial file can be used for, casting one |
| `distribution.md` | Shipping: signing, Gatekeeper, SmartScreen, updates, stores |

A finding earns a line in `docs/rules/` when breaking it would break the player;
it stays in `docs/` when its value is saving the next investigation. **The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [risenxxx/frame-player](https://github.com/risenxxx/frame-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
