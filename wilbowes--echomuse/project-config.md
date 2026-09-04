---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

EchoMuse repurposes Amazon Echo Dot Gen 2 (FireOS 5 / Android 5.1, codename "biscuit") as an open-source voice assistant satellite. Two components:

- **`device/`** — Go binary that runs directly on the rooted Echo Dot
- **`controller/`** — Python asyncio WebSocket server that manages devices, runs wake word detection, and proxies to a voice pipeline
- **`oww_forge/`** — standalone Docker batch trainer for custom openWakeWord models (synthetic TTS positives → augmentation → classifier head → `.onnx`). Not part of the controller; see `oww_forge/README.md`. **Published as an image** since 2026-08-20 (`forge-v*` tags → `forge-release.yml` → `ghcr.io/wilbowes/echomuse-forge`, CUDA on amd64 as `:latest` and CPU multi-arch as `:latest-cpu`) — prefer it to a local build, because the pins below are only preserved by a published artifact. Upstream pins in its Dockerfile are load-bearing (piper-sample-generator v2.0.0 flat layout; openWakeWord SHA with a `--convert_to_tflite` argparse patch). **Extra voices come from `piper_voices.py`, and its catalogue is FETCHED, never hardcoded** — 55 languages, ranked by speaker count, because a baked-in list of English voices makes every other language a code change; the same module backs the phrase preview. `google_tts.py` is rate-limited by Google at any real concurrency, so it retries transient failures and only retires a voice on a permanent refusal. Models install via the dashboard (Config → Wake word → "+ Custom model" → `/api/oww_models/upload`) into `oww_models/` beside the SQLite DB; `owwModel` stores the file path for custom models. openwakeword keys predictions by filename *stem*, never the path — always score via `em_oww_models.prediction_key`

## Where the detail lives

This file holds what is true across both halves. The depth sits in two
directory-scoped files, which load when you touch files in those trees — read
the relevant one before changing anything there.

- **`device/CLAUDE.md`** — building the firmware and the pinned compiler,
  the mic/audio pipeline, on-device wake word and asset distribution, the
  external audio jack, CPU topology and thermals, volume/mute persistence,
  the LED priority system, cgo.
- **`controller/CLAUDE.md`** — running the controller, the Home Assistant
  add-on and release channels, the ESPHome voice backend and HA entities,
  the output chain and ducking, schema migrations, config scoping, activity
  stats, support bundles, OTA, the provisioning wizard, the dashboard.

## Direction: portable, and not dependent on Amazon

**EchoMuse should run on more than one piece of hardware, with minimal change
per platform, and should not depend on Amazon's software to work.** That is
the direction, stated 2026-08-18. It is written here because contributors have
sent multi-thousand-line PRs without knowing which project they were
contributing to, and the answer changes how a change should be judged.

**The dependency is already thin, and keeping it thin is the job.** The entire
Android-specific surface in `device/` is about twenty call sites: `tinymix`
(×10), `stop <service>` (×6), `svc wifi` (×2), `getprop` (×2). Everything else
— mic, speaker, LEDs, buttons, ambient light, jack detect, WiFi state — is
ALSA, i2c, evdev, sysfs and wpa_supplicant. This is a Linux daemon that
happens to be running on Android because that is what shipped on the box.

Three consequences for reviewing a change:

- **Prefer the Linux interface to the Android one**, and where an Android call
  is unavoidable, isolate it rather than spread it.
- **Resolve hardware by NAME, not by number.** `event2` is the volume button
  on biscuit and the *touchscreen* on checkers; opening the wrong one succeeds
  silently and leaves the buttons dead. The same rule already applies to i2c
  (`als.resolve()` matches `tsl2540` by name, since `0-0039` is an
  enumeration accident).
- **A change that makes a vendor blob load-bearing is going the wrong way**,
  and needs to justify itself as a terminal opt-in for one platform rather
  than as the path forward. PR #168 (native AFE) is the **worked example,
  declined 2026-08-21**: opt-in per device, default off, old path untouched,
  built on genuine reverse engineering of the ASP pipeline, and audibly
  better — and still the wrong direction, because it made Amazon's audio HAL
  the path the audio takes. **Decline the direction, keep the findings.** Two
  live bugs it surfaced were fixed on main first (the DAC clipping above
  unity gain, the `Toggle` `disabled` prop), and stock's playback EQ was read
  off a device as coefficients rather than adopted as a binary (#247). We do
  not need Amazon's code to hit Amazon's target, and that is the general
  answer whenever a vendor blob looks like the shortcut.

**LineageOS is probably the wrong target; postmarketOS already has an
`amazon-biscuit` port** (its wiki and pmaports kernel config were corroborating
sources for the ALS second-source diagnosis — see JOURNAL 2026-08-11). There is
no Lineage port for a 2015 MT8163 on Android 5.1, and building one would mean

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilbowes/EchoMuse](https://github.com/wilbowes/EchoMuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
