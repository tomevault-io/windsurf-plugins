---
trigger: always_on
description: **Never commit directly to `main`.** All changes must go through feature branches and be merged via pull request.
---

# Sampler-Sequencer Development Guidelines

## Branching Policy

**Never commit directly to `main`.** All changes must go through feature branches and be merged via pull request.

### Workflow

1. Create a feature branch from `main`:
   ```
   git checkout main
   git pull origin main
   git checkout -b feature/your-feature-name
   ```
2. Make your changes and commit to the feature branch.
3. Push the branch and open a pull request targeting `main`.
4. Merge only after the CI build passes (GitHub Actions runs `flutter build apk --release`).

### Branch naming

Use descriptive prefixes:
- `feature/` — new functionality
- `fix/` — bug fixes
- `chore/` — tooling, dependencies, config changes

## Versioning and Changelog

Every pull request that changes user-facing behaviour **must**:

1. **Update `CHANGELOG.md`** — add an entry under a new version heading (or an `[Unreleased]` section if the version hasn't been decided yet). Follow the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format: `### Added`, `### Changed`, `### Fixed`, `### Removed`.

2. **Bump the version in `pubspec.yaml`** (`version: MAJOR.MINOR.PATCH+BUILD`) following [Semantic Versioning](https://semver.org/):
   - `PATCH` — bug fixes, no new features
   - `MINOR` — new backwards-compatible features
   - `MAJOR` — breaking changes
   - `BUILD` — increment by 1 on every release (Android `versionCode`)

Pure housekeeping changes (CI config, docs, tooling) do not require a version bump, but should still note the change in `CHANGELOG.md` if it affects developers.

## TODO

When completing a task from `TODO.md`, mark it `- [x]` and move it to the **bottom** of its section, but **above** any already-completed `[x]` items there, so the newest completion sits at the top of the completed group.

## Pre-Commit Code Review

Before committing any code change, perform a self-review of the diff. Work through the checklist below and fix any issues found before proceeding with the commit.

### Style

- Dart code follows the [Dart style guide](https://dart.dev/guides/language/effective-dart/style): `lowerCamelCase` for variables/methods, `UpperCamelCase` for types, `_private` prefix for private members.
- No commented-out code, debug prints, or TODO comments left in (unless the TODO is tracked and intentional).
- Consistent formatting — run `dart format` if in doubt.

### Design

- New logic is placed in the correct layer: business logic in `SequencerModel`, DSP/audio in `dsp_utils.dart` or `AudioEngine`, UI-only state in widgets.
- No unnecessary abstractions introduced for a single use case.
- No speculative future-proofing (feature flags, unused parameters, over-engineered inheritance).
- Dependencies flow inward: UI → model → audio; never the reverse.

### Potential Issues

- No new platform-dependent code introduced into otherwise testable units.
- No hard-coded magic numbers without a named constant.
- No state mutation that bypasses `notifyListeners()`.
- No `async` gaps that could leave the UI in a stale state (e.g. awaiting a fire-and-forget call and treating it as complete).
- SharedPreferences keys are unique and follow the existing `_kPrefs*` naming convention.
- No security issues: no command injection, no untrusted input used in file paths or shell commands.
- If any drum generator in `dsp_utils.dart` was modified, `_kPresetCacheVersion` in `audio_engine.dart` has been incremented so the on-device WAV cache is invalidated.

#### AudioEngine invariants (must survive any rewrite of `init()` or `_rebuildPlayer()`)

Every `AudioPlayer` created in `AudioEngine` — sequencer players and the preview player — **must** have these three properties set before use:

| Property | Value | Reason |
|---|---|---|
| `setReleaseMode` | `ReleaseMode.stop` | Prevents `soundPool.release()` on sample completion, which would free the shared SoundPool and silence all other tracks mid-play. |
| `setAudioContext` | `AudioContextAndroid(audioFocus: AndroidAudioFocus.none)` | audioplayers' `FocusManager` requests `AUDIOFOCUS_GAIN` on every `play()` for **all** player modes including lowLatency/SoundPool. When any track triggers it steals focus, sending `AUDIOFOCUS_LOSS` to the other tracks which then stop themselves. Disabling focus management lets all 4 tracks play fully independently. This bug has been reintroduced twice (PRs #30-area and #33). |
| `setPlayerMode` | `PlayerMode.lowLatency` (sequencer) or `PlayerMode.mediaPlayer` (preview/trim) | SoundPool gives ~1 ms trigger latency; mediaPlayer is required for `seek()`. |

If you are touching `init()` or `_rebuildPlayer()`, verify all three are set on every player before committing.

#### Ping-pong retrigger (do not collapse back to one player per track)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianchurchill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
