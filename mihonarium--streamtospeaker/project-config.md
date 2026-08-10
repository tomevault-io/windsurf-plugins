---
trigger: always_on
description: 1. **Research, don't speculate.** When the user reports a problem you don't fully understand (especially Windows / driver / OS-level), reach for `WebSearch` and `WebFetch` *before* proposing fixes. The user has explicitly called out cases where speculation wasted iteration cycles — examples: the "Internal AUX Jack" label, the "Allow apps and Windows to use this device" toggle (which turned out to be the documented `PKEY_AudioDevice_EnableEndpointByDefault` behaviour, not anything I'd guessed). I
---

# CLAUDE.md — instructions for future Claude Code sessions

## Operating principles

1. **Research, don't speculate.** When the user reports a problem you don't fully understand (especially Windows / driver / OS-level), reach for `WebSearch` and `WebFetch` *before* proposing fixes. The user has explicitly called out cases where speculation wasted iteration cycles — examples: the "Internal AUX Jack" label, the "Allow apps and Windows to use this device" toggle (which turned out to be the documented `PKEY_AudioDevice_EnableEndpointByDefault` behaviour, not anything I'd guessed). If the user says "are you sure?", that's a signal to actually look it up.

2. **Verify with cargo check before pushing.** This is a Linux container; build the service with `cd service && cargo check && cargo check --target x86_64-pc-windows-gnu` after any service edit. The Windows cross-check catches GUI / tray / IOCTL issues that the Linux check skips via `cfg(windows)`. Driver edits can't be syntax-checked here (need WDK) — read carefully and trust the user / CI.

3. **PR-per-merge workflow.** This session pushes to a single fixed branch (`claude/stream-speaker-audio-delivery-Utgpx`), and the user merges each PR as a unit. After every push, check `mcp__github__list_pull_requests` for an open PR matching the branch; if none (previous one merged), open a new one via `mcp__github__create_pull_request` with a focused title describing the new commits. Don't let the branch accumulate too many merged-then-orphaned commits without a fresh PR.

4. **Don't claim "fixed" without verifying.** When a fix lands and the user reports it still doesn't work, the first move is `WebSearch` for what *actually* triggers the symptom — not another guess.

## Project: Stream To Speaker

Windows virtual audio device that streams system audio to network speakers — UPnP/OpenHome (Sonos primarily) **and** AirPlay (RAOP / AirPlay 1, plus AirPlay 2 / HomeKit for HomePod). Two parts:

- **`driver/`** — C++ kernel-mode WaveRT/PortCls driver. Single render endpoint, fixed L16 44.1 kHz stereo. Inverted-call IOCTL pattern delivers audio frames to user mode. Includes a separate non-PnP control device for the IOCTLs.
- **`service/`** — Rust user-mode bridge. Default mode is GUI (egui window + tray icon); `--headless` is the CLI mode; `--web` enables the HTTP/JSON API. Talks to the driver via IOCTLs. Discovers speakers via SSDP (UPnP) and mDNS (`_raop._tcp` / `_airplay._tcp`); controls UPnP via SOAP and AirPlay via RTSP; streams PCM either by serving HTTP `audio/wav` (UPnP pull) or pushing RTP/UDP (AirPlay, ALAC). AirPlay code lives in `service/src/airplay/`.

Shared ABI lives in `include/stream_to_speaker_ioctl.h`. Any change to the on-the-wire layout has to be mirrored in `service/src/ioctl_source.rs`.

## Driver versioning

INF DriverVer format: `MAJOR.MINOR.BUILD.REVISION`. Two pieces, two sources:

### Auto: revision (`REVISION` = git commit count)

Bumped on every CI run and every `build-installer.ps1` run, no manual step. Drives:

1. **INF `DriverVer`** (e.g. `1.0.0.42`) — what Windows uses for PnP version comparison and what Device Manager → Properties → Driver shows. From `StreamToSpeaker.vcxproj`: `<TimeStamp>$(DriverVersionPrefix).$(DriverBuildNumber)</TimeStamp>`, with `DriverBuildNumber` injected via `/p:DriverBuildNumber=N`.
2. **`STREAM_TO_SPEAKER_DRIVER_BUILD` in `driver/driver.h`** — runtime identifier, returned via `IOCTL_STREAM_TO_SPEAKER_GET_VERSION`, logged by the service (`StreamToSpeaker driver opened (proto=1 build=N ...)`).

Both get the same N every build, so `1.0.0.42` in Device Manager == `build=42` in the service log == same `.sys` binary. Override with `-DriverBuild N` on the local script for a reproducible value.

**CI driver cache (build.yml):** the kernel-mode driver is cached and only rebuilt when its *source* changes (cache keyed on `hashFiles('driver/**','include/**')`); a hit skips the WDK, the signing-cert generation, and the MSBuild. So `N` is no longer bumped on *every* CI run — it's the commit count at the **last commit that changed the driver**, stamped into the `.sys`/`.inf` when they were built. Still strictly monotonic across driver changes (PnP upgrades unaffected), and `build=N` still uniquely identifies the `.sys` bits — it just stops advancing on service-only commits. The cache stores the `.cer` *alongside* the `.sys`/`.cat`, so the cert the installer ships always matches the cached binaries (a freshly-generated cert per run would not). `build-installer.ps1` (local) is unchanged — still bumps every run. Bump the `v1` tag in the cache key to force a rebuild after changing build flags without touching driver source.

### Manual: prefix (`MAJOR.MINOR.BUILD`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mihonarium/StreamToSpeaker](https://github.com/Mihonarium/StreamToSpeaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
