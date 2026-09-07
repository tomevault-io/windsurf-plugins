---
trigger: always_on
description: Guidance for AI agents (and humans) working on this repo. Read this first; it
---

# CLAUDE.md — Sonority

Guidance for AI agents (and humans) working on this repo. Read this first; it
captures hard-won, non-obvious knowledge that is expensive to re-derive.

## What this app is

**Sonority** is a cross-platform (iOS / Android / macOS) Flutter app that
**unlocks Sonos home-theater / speaker configurations the official Sonos app
refuses to create**, via Sonos' undocumented **local UPnP/SOAP API** (port 1400).
It's a cleaner, focused alternative to *SonoSequencr*.

### Product principle (important)
**Default: do NOT duplicate features the official Sonos app already has** (EQ/bass/
treble editing, volume, grouping, surround-level editing, night sound, Trueplay
*measurement*, …). Every feature should be something the Sonos app **won't** let
users do. (Two things below look like exceptions but aren't duplication: we
*capture+restore* EQ/surround/volume in profiles — never edit them with sliders —
and we *toggle* an already-measured Trueplay calibration the app hides for
unofficial fronts — never measure it. Both are called out again below.) Examples:
- **Dedicated front L/R speakers** on a soundbar (the bar becomes center). ✅ built
- **Mismatched / app-blocked stereo pairs**. ✅ built
- **Zones** — bond 2–16 speakers into one room (full-range L+R, no L/R split),
  including models the app blocks from zones (Play:1 zones fine on hardware).
  NB: this is the Sonos *zone* feature, NOT temporary playback grouping (which
  the app already does and we don't duplicate). ✅ built
- **Config profiles** — snapshot the current unofficial layout + room names and
  re-apply it in one tap (rebuild fronts/surrounds after moving speakers away).
  The validated #1 SonoSequencr request; unique because the Sonos app won't
  recreate a blocked config. ✅ built

**Deliberate exception (softened principle):** full in-app **surround/sub setup**
and **room renaming** DO exist in the official app, but we now do them anyway —
because profiles are only useful if a *complete* HT/stereo setup can be finished
inside Sonority (otherwise you snapshot a half-config and still bounce to the
Sonos app). Justified by "finish a setup in one app, then save it." Keep this the
*only* exception; don't widen it to EQ/volume/grouping/etc.

**Same reasoning extends to profiles capturing EQ/volume (deliberate, narrow):**
a profile can *snapshot each speaker's current EQ (bass/treble/loudness/night/
speech/sub/surround level) and optionally volume* and re-apply them — a
save/restore capability the Sonos app has no equivalent for. This does NOT
duplicate the app because we only **read the live values at snapshot and write
them back on apply** — there are **no EQ/volume editing sliders** in Sonority
(that WOULD duplicate the app). Keep it that way: capture+restore only, never
standalone editing. (`speaker_settings.dart`, two per-profile toggles — EQ, and
volume separately since restoring volume is surprising, so it's opt-in.) **Volume
capture is a wanted feature, not scope creep** — the motivating use case is a
"night mode" profile: snapshot the whole HT with the volume turned down (and
whatever EQ tweaks suit late-night listening), then re-apply the normal profile
in one tap next day. That's a genuine capture+restore capability with no Sonos-app
equivalent; keep it opt-in and capture-only (never a volume slider). The EQ bundle =
bass/treble/loudness + every `GetEQ`/`SetEQ` token (the shared `eqTypes` list;
all Beam-confirmed): NightMode, DialogLevel, SubGain/SubEnable/SubPolarity/
SubCrossover, SurroundLevel/SurroundEnable/SurroundMode/MusicSurroundLevel,
AudioDelay (lip sync), AudioDelayLeftRear/RightRear (surround distance),
HeightChannelLevel. **Gotcha:** the enable tokens are `SubEnable`/
`SurroundEnable` — WITHOUT the trailing "d" of the SCPD state vars
(`SubEnabled` faults 402). NOT exposed locally (so not capturable): volume
limit, spatial music, TV autoplay/disband-on-autoplay, group audio delay, IR.

The app does **no audio processing** — it only issues the bonding/config SOAP
calls the official app blocks. Audio quality comes from the real speakers.

## Toolchain & commands

Flutter is **not on PATH**; this machine uses **fvm, Flutter 3.44.6**:
```
~/fvm/versions/3.44.6/bin/flutter <cmd>
~/fvm/versions/3.44.6/bin/dart run tool/<x>.dart
```
- `flutter analyze` and `flutter test` must stay green before committing.
- **Android build (AGP 9 / Gradle 9 gotchas, cost real debugging):** on AGP 9
  `android.newDsl=true` is the default and breaks the old `android { kotlinOptions {} }`
  block — `build.gradle.kts` uses the new DSL (top-level `kotlin { compilerOptions {} }`,
  Java 17). The Flutter 3.44 migrator adds `newDsl=false`/`builtInKotlin=false` to
  `gradle.properties` for compat; our code works under either. **Jetifier must stay
  OFF** (`android.enableJetifier=false`) — it OOMs on Flutter's jars under AGP 9 and
  nothing here needs it (all deps are AndroidX). The KGP "built-in Kotlin" warnings
  are a *future*-Flutter concern, deferred (blocked on plugins still applying KGP:
  dynamic_color, home_widget, package_info_plus, shared_preferences_android).
  **R8 is on** for release (`isMinifyEnabled`/`isShrinkResources`); keep rules for
  reflection/channel paths (Flutter, our components, home_widget) live in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CasperVerswijvelt/Sonority](https://github.com/CasperVerswijvelt/Sonority) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
