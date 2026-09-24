---
trigger: always_on
description: Working agreement for this repository. Read `docs/ARCHITECTURE.md` before
---

# CLAUDE.md

Working agreement for this repository. Read `docs/ARCHITECTURE.md` before
changing structure, and `docs/ENGINEERING.md` before writing code. This file
is the short version and the trap list.

## What this is

`sunoh.` — a Flutter/Dart Android music app fronting YouTube Music, Gaana and
Saavn behind one interface, plus the music already on the phone, podcasts and
audiobooks. No accounts, no ads, no analytics of any kind. GPL-3.0 (it links
MetrolistGroup's `innertubex`).

Android is the only supported target. The YouTube path is native Kotlin, so
`flutter run` on a real Android device is the only way to exercise it —
web and desktop builds cannot resolve YouTube streams.

## Commands

```sh
flutter pub get
cp env.example.json env.json                 # then fill it in — see below
flutter run --dart-define-from-file=env.json # Android device or emulator
flutter analyze                              # see "Before finishing" below
dart fix --apply                             # mechanical lint fixes
dart format lib/path/you/touched.dart        # NOT `dart format .` — see below
flutter build apk --split-per-abi --release --dart-define-from-file=env.json
```

Endpoints are **not** hardcoded: they come from `env.json` at compile time
(`lib/config/env.dart`), and `env.json` is gitignored so a public repo carries
no private base URL. A build without it still runs — the on-device library and
the YouTube tier need nothing from sunoh-api — but the catalog screens render
their error state. `scripts/release.sh` refuses to build without it.

`flutter test` runs 217 tests: the Android Auto surface (`auto_browse`,
`auto_media_id`), the lyric parsers (`lyrics_parser`) and the reorder index
conventions (`reorder`). The rest of the app is uncovered — see
`docs/ENGINEERING.md` section 9 for the order to extend it in.

## Priorities, in order

1. 60 fps on a mid-range Android phone.
2. A failure in any one feature never takes down playback.
3. The next reader understands why, not just what.

## The rules that get broken most

- **Performance is a design constraint, not a later pass.** No `BackdropFilter`
  over scrolling content. Lists past ~20 items are lazy (`.builder` /
  `.separated` / slivers). Nothing allocates, sorts, parses or formats inside
  `build()`.
- **400 lines per file, 60 per `build()`, 40 per function.** Nineteen files
  currently exceed this; they are listed in `ARCHITECTURE.md` section 9. You
  are not required to fix them, but **leave every file smaller than you found
  it** — if your change adds more than ~40 lines to an over-limit file, extract
  something first. Split along conceptual seams, never by line count.
- **Design tokens only.** Every colour from `SunohColors`, every text style
  from `SunohType`, every corner from `squircleBorder`/`squircleDecoration`/
  `squircleClip`. A raw `Color(0xFF…)` or bare `TextStyle` outside
  `lib/theme/tokens.dart` is a bug. Icons are Solar, except transport controls
  which are Phosphor Fill.
- **Dependencies point downward.** `api/` never imports `state/` or UI.
  `audio/` never imports a screen. `providers/` is wiring only.
- **Navigation is typed.** Use the `SunohNav` extension in `router/router.dart`.
  Never hand-build a path string in a screen.
- **Comments record why, including rejected alternatives.** This codebase's
  comments are its best feature. Match the density. A load-bearing comment is
  code — if the behaviour changes, the comment changes in the same commit.
- **`debugPrint`, not `print`**, for new code. Prefix with the subsystem:
  `[audio]`, `[ytmusic]`, `[downloads]`, `[deeplink]`.

## Traps

Things that look wrong and are not. Do not "fix" these without reading the
comment above them first.

- **`MiniPlayer()` is deliberately not `const`.** A `const` widget let Flutter
  compare identical references and short-circuit subtree reconciliation, which
  broke its subscription to `appStateProvider`.
- **mpv reports a mid-stream network drop as `eof`, not `error`.** The
  premature-EOF branch in `audio_handler.dart` exists for exactly that.
- **`http-header-fields` is written even when empty.** The property is global
  to the player, so a leftover YouTube User-Agent would 403 the next Saavn
  track.
- **`_restoreInProgress` and `_pendingStartPosition` guard real ordering bugs.**
  `prepareQueue` emits a track-change event before mpv loads the file, and
  `openAll(index: N)` loads index 0 first. Removing either guard silently
  corrupts resume position.
- **The router's catch-all `redirect` to `/home` is load-bearing.** Android
  hands `sunoh://playlist/abc` to go_router as bare path `/abc` before the
  deep-link dispatcher sees it.
- **`AppState.position` writes to a `ValueNotifier`, not `notifyListeners()`.**
  That is what keeps the 1 Hz tick from rebuilding every watching screen. Copy
  this pattern for any high-frequency value.
- **`flutter_native_splash` sits in `dependencies`, not `dev_dependencies`,**
  on purpose — see the comment in `pubspec.yaml`. It breaks the release build
  otherwise.
- **`phosphor_icons`, not `phosphor_flutter`.** The latter subclasses
  `IconData`, which Flutter 3.43 made final.
- **Android Auto browse paths must resolve with a COLD cache.** The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afkcodes/sunoh](https://github.com/afkcodes/sunoh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
