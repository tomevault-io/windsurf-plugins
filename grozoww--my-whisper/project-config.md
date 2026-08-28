---
trigger: always_on
description: Working notes for agents and humans on this codebase. `README.md` is what the app is;
---

# CLAUDE.md

Working notes for agents and humans on this codebase. `README.md` is what the app is;
`CONTRIBUTING.md` is how to build it. This is what to know before changing it.

## What this app is

A macOS menu bar dictation tool. Hold a hotkey, talk, and the cleaned-up text is pasted into
whatever field had focus. Everything runs on the Mac by default: speech through NVIDIA Parakeet on
the Neural Engine, cleanup through rules plus Apple's on-device model.

Not sandboxed, on purpose — the Accessibility API cannot reach other apps from inside the App
Sandbox, and pasting into another app is the entire product. Distribution is Developer ID plus
notarization, never the Mac App Store.

## The four rules

These are from `CONTRIBUTING.md` and they are not negotiable. Everything below is downstream of
them.

1. **Never commit a secret.** Keys come from the user at runtime and live in the macOS Keychain.
   Nothing in the build, the tests or CI may require a key.
2. **The app works with zero keys.** Local speech and local cleanup are the default path.
3. **No telemetry, ever.** The only unattended network request is the GitHub release check, and it
   sends nothing about the user. See `UpdateChecker` — the test `sendsNothingIdentifying` is what
   keeps that true.
4. **Keep the build warning-free.** CI fails on a warning. The Swift 6 concurrency warnings in the
   audio path are real defects; that code runs on the audio thread.

And one that follows from them:

5. **Audio never leaves the Mac implicitly.** Choosing a language Parakeet cannot handle does not
   quietly start uploading — it fails with a message naming the switch the user has to turn on.
   `TranscriptionRouter` is the only place that decision is made.

## Commands

```bash
./scripts/version.sh             # what this build is called, and why
./scripts/run.sh                 # build and relaunch
./scripts/run.sh --build         # build only
./scripts/run.sh --test          # unit tests
./scripts/run.sh --check         # what CI runs: warning-free build, tests, dependency audit
./scripts/run.sh --logs          # stream the app's logs at info level
./scripts/run.sh --selftest speech.wav ru   # transcribe a file, no UI or permissions needed
./scripts/audit-deps.sh          # dependency pinning and vulnerability check
./scripts/package.sh             # build a distributable DMG, signed so permission survives
./scripts/release-cert.sh        # once, ever: the certificate every release is signed with
./scripts/screenshots.sh         # redraw docs/images, the README's screenshots
./scripts/make-icon.swift        # redraw the app icon and menu bar glyph into Assets.xcassets

OURWHISPER_SECTION=modes open -a OurWhisper   # open the window on a given screen
```

`--selftest` exists because the interactive path needs Accessibility permission, which a fresh
clone, a CI runner and an automated agent all lack. **If you are an agent and want to know whether
transcription works, this is the command** — not launching the app.

`OURWHISPER_SECTION` exists for the same reason on the UI side: the window is only reachable by
clicking a menu bar icon, which nothing automated can do. Values are the `NavigationSection` raw
values (`home`, `modes`, `vocabulary`, `configuration`, `sound`, `modelsLibrary`, `history`).

`screenshots.sh` is the third of these. It launches the app with `OURWHISPER_SCREENSHOT=<target>`,
which seeds demo data, poses one screen and prints its window number for `screencapture` — see
`ScreenshotMode`. **If you are an agent and want to see what a screen looks like**, this is the
command, and it is cheaper than the throwaway harness. `OURWHISPER_SCREENSHOT_SIZE=880x560` and
`OURWHISPER_SCREENSHOT_SIDEBAR=collapsed` pose the awkward cases — the window at its minimum, and
the screens whose own list becomes the leftmost thing in the window.

## Layout

```
Sources/
  App/          Entry point, AppState, menu bar
  Core/
    Audio/          Capture, device selection, WAV encoding
    DictationController.swift   The record → transcribe → clean → paste → remember loop
    History/        Transcripts and retention
    Hotkey/         CGEventTap and chord matching
    Injection/      Paste into the focused field
    Modes/          Per-context cleanup profiles
    Networking/     HTTPClient seam — the reason cloud code is testable
    Permissions/    Microphone and Accessibility
    Refinement/     Rule cleanup, on-device model, pipeline
    Security/       Keychain
    Settings/       Settings value, store, theme
    Sound/          Feedback sounds, CoreAudio device list
    Storage/        Paths, JSON file store, lenient decoding
    Transcription/  Provider protocol, Parakeet, Soniox, router
    Update/         Release check
    Vocabulary/     Substitution list
  Resources/    Assets.xcassets — the app icon and menu bar glyph, drawn by scripts/make-icon.swift
  UI/           One directory per screen, plus DesignSystem
Tests/          Swift Testing, no network, no key, no permissions
```

The Xcode project uses **synchronized file groups**: a new file under `Sources/` or `Tests/` joins
its target automatically. Never edit `project.pbxproj` to add a file.

## Things that will catch you out


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grozoww/my-whisper](https://github.com/grozoww/my-whisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
