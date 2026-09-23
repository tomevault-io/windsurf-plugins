---
trigger: always_on
description: Reviewer context. This doc drifts from code between edits — check
---

# AGENTS.md — OpenStrap `edge`

Reviewer context. This doc drifts from code between edits — check
`kAlgoVersion` (`lib/compute/derivation_engine.dart`), `schemaVersion`
(`lib/data/db.dart`), and the `version:` line in `pubspec.yaml` directly
rather than trusting a number written here. Where a source comment disagrees
with an implementation, **the implementation wins** — header comments here go
stale (e.g. `lib/compute/substrate.dart`'s file header still describes a
wake-to-wake day model that `calendarDays()` no longer implements; it walks
local midnight to local midnight). The same drift applies to §2's table and
every line-number citation in §3 below — line numbers move on every edit,
symbol names don't; verify against the source, not this doc.

## 1. What this is

A Flutter app for a reverse-engineered WHOOP 4.0 band. **Fully on-device,
local-first**: BLE offload → SQLite → on-device analytics → UI. No backend owns
user data. Network use is limited to OTA update pointers, opt-in
telemetry/Crashlytics, and BYOK LLM calls.

Three sibling repos, strict separation — push work to the right one:
- `OpenStrap/protocol` — bytes: GATT, framing, CRC, opcodes, record decode.
- `OpenStrap/analytics` — metrics: HRV, sleep staging, readiness, strain.
- `OpenStrap/edge` (**this repo**) — flows, BLE link management, storage, UI.

New opcode/record → protocol. New metric → analytics. New screen/flow/table →
edge. A PR implementing a metric inside `edge/lib/compute` is in the wrong repo
unless it is pure orchestration.

## 2. Architecture map (`lib/`, well over 200 files — these five are the biggest by far)

Line counts drift constantly; don't trust a number here, `wc -l` the file.

| file | owns |
|---|---|
| `data/db.dart` | `LocalDb`: schema ladder (`onUpgrade`), all CRUD, coach views |
| `compute/derivation_engine.dart` | `DerivationEngine`, `kAlgoVersion`, day scheduling, isolate offload |
| `state/app_state.dart` | `AppState` ChangeNotifier — BLE↔DB↔UI orchestration |
| `ble/ble_engine.dart` | GATT connect/drain/history-sync state machine |
| `data/local_repository_impl.dart` | read seam: `day_result`/`metric_series` → screen shapes; zero compute on read |

- `ble/` — engine + `ble_state.dart` **pure policies**: `ReconnectPolicy`,
  `SeqAllocator`, `DrainStopEvaluator`, `RecordGate`, `CounterRegressionDetector`,
  `AckRetryPolicy`, `ChunkFailureLedger`, `DeriveDebouncer`, `AlarmPayloads`,
  `AlarmConfirmation`.
- `sync/` — `sync_policy.dart` **pure policies**: `ClockRef`/`ClockPolicy`,
  `BackfillPolicy`, `MarginalRadioDetector`, `FrameCorruptionDetector`,
  `PostBondTimeoutLoopDetector`, `BondRefusalGiveUp`, `EmptySyncTracker`,
  `StuckStrapDetector`; plus background/headless entries and OTA.
- `compute/` — `substrate.dart` (**single** raw→`Substrate` decode point +
  `calendarDays()` day model), `onehz_pipeline.dart` (pure, isolate-safe per-day
  pipeline), `crossday_pipeline.dart`, `derivation_engine.dart`.
- `data/` — `db.dart`, read seam, `day_label.dart` (the *only* day-label helper).
- `notify/` — `notification_center.dart` is the **single emitter**;
  `fired_keys.dart` is the persistent fire-once guard.
- `coach/` — read-only SQL over allow-listed `v_*` views behind a deny-list guard.
- `ui2/` — 66 files (`lib/ui` was deleted in the UI rebuild): `ui2/theme.dart`
  and `ui2/grammar.dart` (design system), `ui2/charts.dart`, `ui2/screens/`
  (shared metric/trend IA), plus `ui2/onboarding/`, `ui2/activity/`,
  `ui2/profile/`.
- Also `ai/` (BYOK), `gps/`, `health/` (HealthKit/Health Connect export),
  `telemetry/` (opt-in), `widget/` (App-Group snapshot for WidgetKit/watch).

**Storage.** Durable ledger: `decoded_onehz` (1 Hz, `UNIQUE(rec_ts)`,
INSERT-OR-REPLACE) + `decoded_rr` (beats, cascades on eviction) + `raw_archive`
(never pruned; undecodable/unknown-version records) + `raw_records` (retained as
replay/debug ledger and upgrade fallback) + `events`/`band_events`. Derived
output: versioned **immutable** `day_result` (PK `day_id, algo_version`) and
`metric_series` (PK `date,key`, REPLACE).

**Bug-density hotspots** (fix-titled commit churn, last 300 commits):
`state/app_state.dart` 30 · `data/db.dart` 25 · `compute/derivation_engine.dart`
24 · `data/local_repository_impl.dart` 17 · `ble/ble_engine.dart` 12 ·
`main.dart`+`app.dart` 19. Treat diffs in these with extra scrutiny.
`pubspec.yaml` has high raw churn but most of it is release version bumps — not
a hotspot.

## 3. Hard invariants — violating these is a P0 regression

1. **Commit before ACK.** In the history-sync drain (`ble/ble_engine.dart`)
   decoded rows + cursor commit in one transaction *before*
   `buildHistoryResultOk` echoes the verbatim 8-byte HISTORY_END token. The band
   trims flash on ACK. Reordering, or echoing a regenerated/mangled token, causes
   permanent data loss or an infinite re-flood. Never ACK a partial chunk.
2. **`decoded_onehz` stays INSERT-OR-REPLACE keyed on `rec_ts`.** INSERT-OR-IGNORE
   breaks counter-reset recovery. Evicting a row must delete that counter's
   `decoded_rr` beats in the same batch.
3. **Never fabricate a metric.** Absent input ⇒ null / `Metric.absent` / "—". No
   imputation, no substituted defaults, no deriving one metric from another as a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenStrap/edge](https://github.com/OpenStrap/edge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
