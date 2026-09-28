---
trigger: always_on
description: This file is the implementation guide for future agents working in this repository.
---

# AGENTS.md

This file is the implementation guide for future agents working in this repository.

Read this before adding a new feature or extending an existing metric screen.

## Purpose

The app is moving toward a consistent, period-based detail architecture for health metrics.

The goal is:

- dashboard-first navigation
- feature-first code organization
- clear separation between data access, feature state, and UI
- reusable screen scaffolding without forcing all metrics into one generic chart system

## Source Of Truth

Use these docs together:

- [docs/README.md](docs/README.md): doc index
- [docs/engineering/architecture.md](docs/engineering/architecture.md): target architecture, package map, and the app-wide cross-cutting rules
- [docs/engineering/feature-playbook.md](docs/engineering/feature-playbook.md): step-by-step guide for adding a feature, a settings section, a Room entity, or a device integration
- [docs/engineering/development.md](docs/engineering/development.md): build and verification tasks, including the known translation-gate failure
- [docs/engineering/test-parity/README.md](docs/engineering/test-parity/README.md): Flutter ↔ Kotlin test parity matrix and outstanding gaps
- [docs/engineering/analysis/README.md](docs/engineering/analysis/README.md): code analysis (MVVM, Clean Architecture, Compose performance, refactor backlog)

If code and docs disagree, prefer the docs for new work and refactor toward them incrementally.

## Current State

The codebase already has aligned period-based detail screens for:

- steps/activity
- sleep
- heart
- activities
- body

The global Browse feature has been removed. Entries and sessions should be browsable from the relevant dashboard widget/detail screen instead of through a standalone app destination.

These features already show the intended direction.

Beyond the metric screens, the app now carries three subsystems that are not metric features and do not follow the period-detail pattern:

- `devices/` — the device layer: the Garmin GFDI protocol stack, the shared BLE radio lease, companion-device pairing, and notification forwarding. `features/watches` is its UI.
- `features/devicesync/` — phone-to-phone Health Connect sync over Bluetooth Classic RFCOMM.
- `data/migration/` — a one-time Flutter-to-Kotlin data importer that runs in two phases from `OpenVitalsApp.onCreate()`. Its ordering around `super.onCreate()` is load-bearing; read the architecture doc before touching startup.

The following areas are still transitional and should not be copied as the default pattern:

- duplicated period selection logic in multiple ViewModels
- broad shared component files that still mix several concerns
- oversized screen files that should keep being split into route, section, card, and helper files

## Golden Path For New Metric Features

When adding a new detail feature, follow this shape:

1. Define the feature contract.
   - screen state
   - user actions
   - any derived display fields

2. Make the feature period-driven.
   - support `Day / Week / Month / Year`
   - use a selected anchor date
   - support previous/next navigation
   - cap navigation at the current period

3. Keep the frame reusable, keep the charts specific.
   - reuse the shared period scaffolding
   - keep metric-specific cards and charts inside the feature package

4. Keep repository APIs query-oriented.
   - prefer `DatePeriod` or feature query objects over adding more ad hoc overloads
   - keep Health Connect specifics below the feature layer

5. Register the feature from the dashboard.
   - dashboard card
   - route
   - top bar title

6. Update docs if the pattern evolves.

## Invariants

Do not break these without an explicit decision. They are app-wide, and each has a section in [docs/engineering/architecture.md](docs/engineering/architecture.md).

- **No `INTERNET` permission.** The manifest removes `INTERNET`, `ACCESS_NETWORK_STATE`, and `ACCESS_WIFI_STATE`. Phone-to-phone sync is Bluetooth Classic specifically so this stays true. Never add a dependency that needs a socket.
- **One foreground service at a time.** Activity recording, the Apple Health import, and phone sync contend for the single foreground slot and refuse rather than queue.
- **One BLE radio, leased per address.** Everything that opens a BLE link takes a lease from `devices/core/RadioLease.kt` under one of the four owner tags: `SYNC`, `FIND`, `SETTINGS`, `NOTIFICATIONS`. A lease is re-entrant per tag, so `SYNC` work (a sync, a file upload) also serialises on `GarminWatchSyncService.syncMutex`.
- **A missing permission is `ScreenError.PermissionDenied`.** Use `isPermissionFailure()` / `toScreenError()`; never pattern-match exception messages. The screens render this as a grant affordance.
- **Health Connect reads and record mapping live behind `healthconnect/*HealthReader`.** Writes go through `AppleHealthImportRepository.insertImportedRecords` with a deterministic `clientRecordId`.
- **Nothing waits on the main thread.** No `runBlocking` in `app/src/main` (`NoRunBlockingRatchetTest` holds the allow-list). A receiver never holds a broadcast for a Health Connect read. Composables `remember` any pass over samples.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenVitals-mtu/OpenVitals](https://github.com/OpenVitals-mtu/OpenVitals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
