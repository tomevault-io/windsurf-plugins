---
trigger: always_on
description: > This file is auto-loaded each session. It is the end-to-end orientation for
---

# CLAUDE.md — OpenHearing project context

> This file is auto-loaded each session. It is the end-to-end orientation for
> anyone (human or AI) picking up this repo. Full chronological history is in
> [docs/PROJECT_LOG.md](docs/PROJECT_LOG.md).

## What this is

**OpenHearing** — a free, open-source Android app bringing AirPods Pro 2/3-style
hearing assistance to Android: (1) a pure-tone hearing **screening** → audiogram,
(2) turn it into an amplification/EQ **profile**, (3) **assist** mode amplifies
quiet sound (speech) in real time on any earbuds.

**Framing is load-bearing:** this is a *hearing-assistance / sound-amplification
tool, NOT a medical device*. Never use "diagnose / treat / medical" in user-facing
copy. Every health surface carries the disclaimer (README, onboarding, before any
test). Hearing safety is non-negotiable — see [docs/SAFETY.md](docs/SAFETY.md).

- **Repo:** https://github.com/HMAKT99/OpenHearing (owner `HMAKT99`, branch `main`)
- **License:** GPLv3 · **Author:** Arun Kumar Thiagarajan <arunkt.bm14@gmail.com>

## Current status (2026-07-03)

Phases 0,1,2,4,5 plus consumer phases A,B are **built, unit-tested**. The app is
**alpha — ready for hardware testing, NOT ready to ship to consumers.**

| Phase | State |
|---|---|
| 0 Scaffold (modules, CI, docs, license) | ✅ done |
| 1 Audiogram engine (staircase, fitting) + check screen | ✅ done |
| 2 Real-time assist DSP + Android engine + service | ✅ done |
| 3 AirPods protocol | ❌ **not started** — UNVERIFIED, [docs/PROTOCOL.md](docs/PROTOCOL.md) |
| 4 Persistence, onboarding, assist UI, accessibility | ✅ done |
| 5 Release build, signing, privacy, F-Droid metadata | ✅ done |
| A Consumer polish (icons, results chart, translatable strings, regulatory copy, About, fastlane assets) | ✅ done 2026-07-02 |
| B User-demanded features (live gain, manual audiogram entry, multi-profile history, disconnect auto-stop + speaker warning, QS tile) | ✅ done 2026-07-03 |
| C Differentiators (per-ear stereo assist, environment presets, experimental media EQ) | ✅ done 2026-07-03 |
| Calibration | ✅ comfort-ceiling proxy; true dB SPL needs a meter |

**88 unit tests green; debug + release (R8) APKs build.** UI flows exercised on
an API 35 emulator (screenshots in fastlane metadata). Market research + launch
playbook and the FDA "audiogram + fitting formula = device design" finding are
recorded in [docs/PROJECT_LOG.md](docs/PROJECT_LOG.md) (Phase A entry).

### TWO GATES before any consumer release (cannot be closed in code)
1. **On-device safety validation** — no audio has ever run on a real device. Assist
   mode (live mic, feedback guard, real earbuds, latency) and the limiter must be
   verified on hardware per [docs/DEVICE_TESTING.md](docs/DEVICE_TESTING.md).
2. **Real calibration + legal/regulatory** — comfort calibration is a safe proxy,
   not dB-SPL truth; medical-device framing/jurisdiction needs the maintainer's
   judgement (no legal advice from here).

## Key decisions (don't re-litigate without reason)

- **Kotlin, Compose/Material 3, MVVM + Hilt, coroutines/Flow.** minSdk 26,
  compile/target SDK 35, JDK 17.
- **DSP is pure Kotlin** behind I/O interfaces (AAudio/AudioTrack/AudioRecord is a
  thin shell) so the safety-critical and signal logic is JVM-unit-tested.
- **Namespace / applicationId:** `app.openhearing`.
- **Fitting = half-gain rule** for v1 (uncalibrated + no per-band yet); NAL-NL2
  slots behind `FittingStrategy` later — see [docs/FITTING.md](docs/FITTING.md).
- **Earbud-agnostic first.** Works fully on any headset; AirPods is a best-effort,
  clearly-`UNVERIFIED` enhancement, never a hard dependency.
- **Assist is per-ear stereo** since Phase C: mono mic in, stereo out, one chain
  (curve + limiter) per channel. Media EQ is experimental (deprecated
  global-session effect; may not work on all OEMs).

## Module map (see [ARCHITECTURE.md](ARCHITECTURE.md))

- `:core-common` — units (`Hertz`, `DecibelsHl/Spl/Fs`, `Ear`) + **`SafetyConstants`**
  (single source of truth for output limits).
- `:core-audiogram` (pure JVM) — `Audiogram`, `HughsonWestlakeStaircase`,
  `PureToneScreening`, `GainCurve`/`FittingStrategy`/`FractionalGainRule`, `AudiogramCodec`.
- `:core-audio` (Android lib) — `dsp/`: `Biquad`, `GainEqualizer`, `Wdrc`,
  `FeedbackGuard`, `LookaheadLimiter`, `HearingAssistChain`; `ToneGenerator`,
  `TonePlayer`, `AndroidAudioEngine`, `OutputLimiter`.
- `:airpods-protocol` (Android lib) — `AirPodsController` interface, all `UNVERIFIED`.
- `:data` (Android lib) — `SettingsRepository` + `ProfileRepository` (DataStore;
  multi-profile list via internal `ProfileListCodec`, legacy keys auto-migrate).
- `:app` — Compose UI (`OpenHearingApp` nav, onboarding gate, hearingtest/ incl.
  `AudiogramChart`, manualentry/, assist/, settings), Hilt modules (`di/`),
  `assist/AssistController` + `AssistSessionFactory` + `AssistService` +
  `AssistTileService` (quick-settings tile).

Data flow: screening → `Audiogram` → `FittingStrategy` → `GainCurve` →
`HearingAssistChain` (EQ→WDRC→feedback guard→master gain→**limiter**) →
`AndroidAudioEngine`. The limiter is ALWAYS the final stage.

## Safety invariants (treat violations as critical bugs)

- Nothing reaches the device without passing an `OutputLimiter`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HMAKT99/OpenHearing](https://github.com/HMAKT99/OpenHearing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
