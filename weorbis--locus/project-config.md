---
trigger: always_on
description: - **Framework**: Flutter
---

# locus

## Tech Stack

- **Framework**: Flutter
- **Language**: Dart
- **Package Manager**: flutter pub

## Commands

```bash
# Get dependencies
flutter pub get

# Build
flutter build

# Test
flutter test

# Run
flutter run

# Analyze
flutter analyze
```

## Project Structure

Locus is a **Flutter plugin** (not an application) — a background geolocation SDK with native Android/iOS implementations and a minimal Dart facade.

```
locus/
├── lib/                          # Dart SDK (public API)
│   ├── locus.dart                # Single entry point — barrel re-exports only
│   └── src/
│       ├── core/                 # Platform channels, abstract interface, event streams, lifecycle
│       ├── config/               # Config, presets, validators, enums, constants
│       ├── features/             # Feature-first modules, each with models/ + services/
│       │   ├── location/         # Core tracking, quality analysis, spoof/anomaly detection
│       │   ├── geofencing/       # Circular + polygon geofences, workflow engine
│       │   ├── battery/          # Adaptive tracking, runway estimation, power state
│       │   ├── privacy/          # Privacy zones (exclude / obfuscate / reduce)
│       │   ├── trips/            # Trip detection, route recording, persistent trip store
│       │   ├── sync/             # HTTP queue, batch sync, retry, connectivity handling
│       │   ├── tracking/         # Tracking profiles, rule-based profile switching
│       │   └── diagnostics/      # Logging, debug overlay widget, error recovery
│       ├── services/             # Service interfaces + default implementations
│       ├── shared/               # Cross-cutting models (Coords, Activity, Battery, …)
│       └── testing/              # MockLocus — for host-app tests
├── android/src/main/kotlin/dev/locus/
│   ├── LocusPlugin.kt            # FlutterPlugin entry; wires method + event channels
│   ├── core/                     # ConfigManager, LocationTracker, StateManager, TrackingLifecycleController
│   ├── location/                 # FusedLocationProvider client wrapper
│   ├── activity/                 # MotionManager (ActivityRecognitionClient)
│   ├── geofence/                 # GeofencingClient bindings
│   ├── receiver/                 # Boot, notification action, geofence, activity broadcast receivers
│   ├── service/                  # ForegroundService, HeadlessService, HeadlessValidationService
│   └── storage/                  # Persistent queue, SharedPreferences wrappers
├── ios/Classes/                  # Swift + ObjC plugin (CLLocationManager, CMMotionActivityManager)
├── bin/                          # Dart CLI executables — setup, doctor, migrate, locus
├── test/                         # unit/ integration/ benchmark/ fixtures/ helpers/ mocks/
├── doc/                          # guides/ core/ reference/ advanced/ setup/ api/ testing/
└── example/                      # Example Flutter app consuming the plugin
```

Host apps import **only** `package:locus/locus.dart`. Everything under `lib/src/` is an implementation detail and not part of the semver contract. The `bin/` tools (`setup`, `doctor`) are declared as `executables:` in `pubspec.yaml` and run via `dart run locus:<tool>`.

## Code Style & Conventions

- Follow Dart style guide and effective Dart
- Use `dart format` for code formatting
- Prefer Riverpod for state management (if applicable)
- Use freezed for immutable models (if applicable)
- Separate presentation, domain, and data layers

## Key Dependencies

**Runtime** (see `pubspec.yaml`):

- `permission_handler` — Runtime prompts for location (fine, coarse, background), notifications, activity recognition, and battery-optimization exemption.
- `device_info_plus` — OS/device metadata (SDK level, manufacturer, model) used by `PermissionAssistant` and diagnostics to route around OEM quirks (Samsung, Xiaomi, Huawei power management).
- `http` — HTTP client backing the sync queue in `lib/src/features/sync/`.
- `uuid` — Stable IDs for queue items, geofence registrations, trip sessions, and log entries.
- `logging` — Structured `Logger` tree exposed through `LocusDiagnostics` and the debug overlay.
- `args` — Argument parsing for CLI executables (`bin/setup.dart`, `bin/doctor.dart`, `bin/migrate.dart`).

**Dev / test**:

- `flutter_test` — Widget + unit test harness.
- `flutter_lints` — Baseline lint set referenced by `analysis_options.yaml`.

**Native dependencies** (declared in `android/build.gradle` and the iOS podspec — relevant when changing platform code):

- Android: `play-services-location`, `play-services-activity-recognition`, `androidx.work`, `androidx.core`.
- iOS: `CoreLocation`, `CoreMotion`, `UserNotifications`.

The SDK deliberately keeps its Dart dependency surface **minimal** — no Riverpod, no freezed, no code-gen. All models are hand-written immutable data classes. This keeps plugin install size small, compile times fast, and avoids dragging host-app state-management opinions into an SDK. **Do not add runtime Dart dependencies without explicit approval** — every one becomes a transitive dependency for every consuming app.

## Quality Standards

- Quality over speed — always
- Spec before code for non-trivial changes (use the design-spec skill)
- Write or update tests alongside every change

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weorbis/locus](https://github.com/weorbis/locus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
