---
trigger: always_on
description: Guidance for AI coding agents working on SlyTerm. Human contributors: the same rules are explained
---

# AGENTS.md

Guidance for AI coding agents working on SlyTerm. Human contributors: the same rules are explained
in [CONTRIBUTING.md](CONTRIBUTING.md).

SlyTerm is a macOS menu bar app, one Swift package (`Package.swift`, Swift 5.9, macOS 14): a
terminal that floats over games, with click-through, global hotkeys, coding agent session status and
answers (Claude Code, Codex, omp, pi, and Gemini CLI and Qwen Code from their titles), web tabs that show guides and video and can float over the game, an OCR lookup that opens a
game's wiki in one, and bringing sessions in from other terminals. Its dependencies are SwiftTerm
and Sparkle, for updates.
Behaviour and architecture are in [docs/TECHNICAL.md](docs/TECHNICAL.md); the
[source map](docs/TECHNICAL.md#source-map) says which file holds what.

## Commands

```sh
swift build                          # the build check; binary at .build/debug/SlyTerm
./build.sh                           # app bundle at dist/SlyTerm.app, signed ad hoc, no updates
./build.sh --release                 # signed, notarized app and DMG in dist/: only when asked
.build/debug/SlyTerm --strip-snapshot /tmp/strip.png   # an offline check (full list below)
```

Signing, notarizing and publishing are in
[Building and releasing](docs/TECHNICAL.md#building-and-releasing).

There is no test target. A change is verified with the binary's command-line modes, which run
offscreen and never type into anything:

| Area | Modes |
| --- | --- |
| Lookup | `--match <text>`, `--ocr <png> [--at X Y] [--pick] [--fast]`, `--lookup [x y]`, `--search <text>`, `--index [--refresh]`, `--probe <url>`, all with `--game <name>` or `--preset <name>` |
| Pick mode | `--pick-snapshot <png> X Y <out.png> [--scale 2]` |
| Web tabs | `--guide-snapshot <url> <out.png> [--full] [--fill] [--width N] [--height N] [--scroll N] [--eval <js>] [--find <text>]`, `--float-snapshot <out.png>`, `--drm-check` |
| Tab strip | `--strip-snapshot <out.png>` |
| Activity | `--activity [--json]`, `--activity --transcript <file.jsonl> [--status waiting]`, `--activity --title <text>`, `--activity --screen <file>`, the last three with `[--agent <name>]`, `--activity --poll [--times N]`, `--card-snapshot <out.png>`; fixtures from `swift Tools/make-agent-fixtures.swift <dir>` |
| Bring In a Session | `--sessions [--json] [--session <uuid> \| --pid <pid> \| --tty <tty>] [--send-back [--copy]]`, `--picker-snapshot <out.png>` |
| Setup assistant | `--setup-snapshot <out.png> [--step N]` |

`swift Tools/make-lookup-fixtures.swift <dir>` draws synthetic game screenshots and prints the
`--at` coordinates and the expected first candidate for each. When a mode writes a PNG, open it
and look at it: for drawing changes that is the test.

CI (`.github/workflows/ci.yml`) runs `swift build`, the strip, float, card, picker and setup
snapshots, and pick mode over every lookup fixture on each pull request, and attaches the PNGs to
the run as `snapshots`. It checks that they were written, not what they show. Its App bundle step
runs `./build.sh`, verifies the signature and runs a snapshot from the bundle's binary, which fails
if the bundle cannot load Sparkle. A pull request that touches the release inputs (the `paths` in
`.github/workflows/release.yml`: `build.sh`, `Info.plist`, `Package.swift` and the like) also gets
a dry run of the Release workflow, signed and notarized but never published.

## Hazards

Read these before running anything.

- **Any argument that is not one of the modes above launches the full GUI app.** There is no
  `--help`. A stray launch puts a window on the user's screen, writes preferences and
  `~/Library/Application Support/SlyTerm/`, starts a shell for every restored tab and may run the
  user's `startupCommand`, which is often `claude`. Pass only the flags listed.
- **The user's real app may be running.** Check `pgrep -xl SlyTerm` before a live launch, and do
  not quit it without asking. `make run` quits every running SlyTerm, and `./build.sh --install`
  overwrites `/Applications/SlyTerm.app`: run neither unless asked.
- **Preferences are shared.** `dist/SlyTerm.app` and its binary use the installed app's
  `com.charlesmelki.slyterm` domain; `.build/debug/SlyTerm` uses its own `SlyTerm` domain. Before a
  live run of the bundle, `defaults export` the domain to a backup, and afterwards `defaults
  delete` it and `defaults import` the backup.
- **Releases use the owner's identity.** `./build.sh --release` signs with the owner's Developer
  ID and submits to Apple's notary service: run it only when asked, and use `NOTARIZE=0` to check
  signing without submitting. `Tools/make-appcast.sh` without `--ed-key-file` reads the update key
  from the login keychain and puts a keychain dialog on the user's screen. Never dispatch the
  Release workflow, publish a release, push a tag or push to `MushkyQT/homebrew-tap` (its
  publish job updates the cask there) unless asked. A release-built copy launched
  live checks the real feed: point `SLYTERM_FEED_URL` at a local feed for tests.
- **`open slyterm://…` reaches the registered copy,** usually the installed one. Target the build
  under test with `open -g -a dist/SlyTerm.app "slyterm://…"`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MushkyQT/SlyTerm](https://github.com/MushkyQT/SlyTerm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
