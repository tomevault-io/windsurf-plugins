---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is — and what it isn't

This is the **integration surface** of Locus Map, not the app. It's the contract that third-party
Android apps depend on to read state from Locus, push data into it, or expose a sensor. No map
rendering lives here; nothing in `locus-core` (the actual Locus Map / GIS apps) lives here. Treat
every public class, AIDL interface, Parcelable field, manifest constant, and serialised byte
layout as an external contract — breaking it breaks every third-party app that targets a given
`API_CODE`. Bump versions and update [`CHANGELOG.md`](CHANGELOG.md) on every surface change.

The app-side counterparty (Locus Map / Locus GIS) lives in a separate, private repository.
Conventions from there — Compose, Hilt, Room, clean-architecture layering, library codes,
news-file changelogs — **do not apply here**. This repo is a small, focused library with its
own rules; the topical guides below replace any inherited convention.

## Topical guides — read before working in these areas

When the current task touches one of the rows below, **read the linked file before writing code**.

| Working on… | Read first | Hard rule (summary) |
|---|---|---|
| Sensor adapter framework, manifest XML schema, AIDL contract, refIds | [`docs/android/guides/adapter-apps/`](docs/android/guides/adapter-apps/) | Contract version is [`AdapterApi.VERSION`](locus-api-android/src/main/java/locus/api/android/features/sensorAdapter/AdapterApi.kt) — bump **only** on breaking AIDL or manifest changes. Locus owns BT3/BT4/USB transport; adapter parses bytes via `parseData(deviceId, source, bytes)`. |
| Curated `LocusVariable` refIds (HeartRate, Cadence, Power, …) | [`docs/android/reference/locus-variables.md`](docs/android/reference/locus-variables.md) | Catalogued refIds are stable IDs Locus consumes — adding/renaming requires CHANGELOG entry and matching update on the Locus Map side. |
| Routing / compute-track provider (`ComputeTrackService`, `IComputeTrackService`, route types) | [`docs/android/guides/routing-apps/`](docs/android/guides/routing-apps/) | No contract-version constant — compatibility rests on append-only `Storable` rules for `ComputeTrackParameters` / `Track`. Params/Track ride as `Storable` bytes inside a `ParcelableContainer`, so the AIDL shape never changes. Build routes as `Track.points` (geometry) + `Track.waypoints` (instructions linked by `parameterRteIndex`). |
| Modifying `Storable` subclasses (Point, Track, GeoData, GeocachingData, …) | [`locus-api-core/src/main/java/locus/api/objects/Storable.kt`](locus-api-core/src/main/java/locus/api/objects/Storable.kt) | `getVersion()` is monotonic; **never** remove or reorder fields in `readObject` / `writeObject`. Add new fields only after the current version's last field, behind a `version >= N` check on read. Old clients must still parse new payloads. |
| AIDL interfaces under `locus-api-android/src/main/aidl/**` | [`docs/android/guides/adapter-apps/aidl-contract.md`](docs/android/guides/adapter-apps/aidl-contract.md) (sensor adapter) | AIDL method ordinals are part of the wire format — only append new methods; never reorder or remove. Use new methods on a new interface version when behaviour changes. |
| Intent / extra constants, `LocusConst`, action keys | [`locus-api-android/src/main/java/locus/api/android/utils/LocusConst.kt`](locus-api-android/src/main/java/locus/api/android/utils/LocusConst.kt) | Constants are wire-format strings consumed by released app builds — don't rename, only add. |
| Adding a new "Action…" entry point (point/track display, recording control, etc.) | [`locus-api-android/src/main/java/locus/api/android/ActionBasics.kt`](locus-api-android/src/main/java/locus/api/android/ActionBasics.kt) | Co-locate with related actions under `locus.api.android`; document required `VersionCode` and update CHANGELOG. |

## Build & version

This is a Gradle multi-project with two library modules plus two sample apps.

```bash
# Build everything (libraries + samples)
./gradlew clean assembleDebug

# Library-only builds
./gradlew :locus-api-core:build
./gradlew :locus-api-android:assembleRelease

# Sample apps
./gradlew :samples:android-api-explorer:assembleDebug
./gradlew :samples:android-sensor-adapter:assembleDebug

# Publish to local Maven (for testing against locus-core without a JitPack round-trip)
./gradlew :locus-api-core:publishToMavenLocal :locus-api-android:publishToMavenLocal
```

**No test suite ships with this repo.** There is no `:test` task and no `src/test/` tree —
verification is done downstream (in locus-core's tests and in the sample apps). Don't add
ceremony tests "for the sake of CI"; if a behaviour is worth testing, the right place is
locus-core's instrumentation suite, which actually exercises the IPC.

**Versions** live in [`gradle.properties`](gradle.properties):

| Property | Meaning |
|---|---|
| `API_CODE` | Monotonic int. Bump on **every release** — third-party apps gate features on `>= N`. |
| `API_VERSION` | Semver string used as the published Maven version and JitPack tag. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asamm/locus-api](https://github.com/asamm/locus-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
