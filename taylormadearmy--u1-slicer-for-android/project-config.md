---
trigger: always_on
description: Android app wrapping **Snapmaker Orca 2.2.4** (OrcaSlicer fork) for Snapmaker U1 (270×270×270mm, 4 extruders).
---

# U1 Slicer for Android

Android app wrapping **Snapmaker Orca 2.2.4** (OrcaSlicer fork) for Snapmaker U1 (270×270×270mm, 4 extruders).
Kotlin + Jetpack Compose + Material3 blue theme + Native C++ via JNI.
App ID: `com.u1.slicer.orca`
Current release: `v1.5.65` (`versionCode 231`)

> For local-only device IDs, adb targets, and any machine-specific workflow notes, see `CLAUDE.local.md` if present.
> For the current deep-dive on the post-upgrade native Clipper failure, see [`CLIPPER_UPGRADE_INVESTIGATION.md`](CLIPPER_UPGRADE_INVESTIGATION.md).

## Build

```bash
./gradlew installDebug          # Build and install on connected device
./gradlew assembleDebug          # Build APK only
```

Gradle daemon may OOM — use `--no-daemon` if builds fail.

## Release

1. **Bump version** in `app/build.gradle` - increment both `versionCode` and `versionName` (e.g. `1.4.10` -> `1.4.11`)
2. **Update docs** — update test counts in this file and `README.md` if they changed
3. **Commit and push**:
   ```bash
   git add -p
   git commit -m "bump: v1.4.11 - <short description>"
   git push
   ```
4. **Build the release APK**:
   ```bash
   ./gradlew assembleRelease --no-daemon
   ```
5. **Rename the APK** with the version number:
   ```bash
   cp app/build/outputs/apk/release/app-release.apk u1-slicer-v1.4.11.apk
   ```
6. **Create a GitHub release** (never overwrite or delete an existing release — always use a new tag):
   ```bash
   gh release create v1.4.11 u1-slicer-v1.4.11.apk \
     --title "v1.4.11" \
     --notes "Brief description of what changed."
   ```

> **Rule**: Never reuse or update a published GitHub release. If you need to fix something, bump to a new version.

## Security

Public vulnerability reports should follow [`SECURITY.md`](SECURITY.md). Keep any private device IDs, adb targets, and local test notes in `CLAUDE.local.md` only.

## Test

```bash
./gradlew testDebugUnitTest                        # 802 JVM unit tests
./gradlew connectedDebugAndroidTest                # 168 instrumented tests (uses Orchestrator)
```

For local device IDs and any private E2E notes, consult `E2E_TESTING.local.md` if present.

> **All tests must pass — there are no known pre-existing failures.** If a test fails, investigate it; do not assume it is a pre-existing or flaky issue.

### Unit tests (`app/src/test/`) - 802 tests across 54 classes
- `gcode/GcodeParserTest.kt` (33) — G-code parsing: layers, extrusion, extruder switching, ;TYPE: feature-type tagging, wipeTowerFilamentMm, B52 maxMoves cap + stride distribution
- `gcode/GcodeValidatorTest.kt` (45) — Tool changes, nozzle temps, layer count, prime tower footprint, bed bounds validation
- `gcode/SuspiciousLineContextTest.kt` (6) — B52 streaming line context lookup: window clamping, multi-sample cap, large file smoke test
- `gcode/GcodeToolRemapperTest.kt` (19) — Compact tool index remapping, SM_ params, M104/M109
- `viewer/StlParserTest.kt` (10) — Binary/ASCII STL parsing, bounding box, vertex data, 10-float vertex format
- `viewer/MeshDataTest.kt` (11) — MeshData 10-float vertex format, extruderIndices, recolor(), RGBA values, multi-extruder recolor
- `viewer/ThreeMfMeshParserTest.kt` (29) - 3MF mesh parsing, per-triangle color extraction, extruderMap, MeshWithContext, SEMM paint_color parsing, multi-object extruder map
- `network/MakerWorldUtilsTest.kt` (36) — URL parsing, design→instance ID resolution, download response parsing, error classification, cookie sanitization
- `network/MoonrakerClientTest.kt` (36) — PrinterStatus computed properties, URL normalization, LED state, remoteScreenUrl(), B33 virtual_sdcard progress parsing, sendGcode network path
- `data/SliceConfigTest.kt` (25) — Default values match Snapmaker U1 hardware specs, wipe tower bounds clamping
- `data/DataClassesTest.kt` (17) — FilamentProfile, SliceJob, GcodeMove, ModelInfo, WipeTowerInfo
- `data/PlateTypeTest.kt` (21) — PlateType.bedTempFor per-material presets, fromName, case-insensitivity
- `data/SlicingOverridesTest.kt` (85) — Override modes, JSON serialization round-trip, defaults, resolveInto(), multi-extruder wipe tower, B24 stale config, B31 brim_type, F30/F31 plus F41/F42/F43 override/file-value coverage, F57/F58 primeTowerWidth + wipeTowerRotationAngle, B53 computeTogglePrimeTower, B71 nozzle temp extruderTemps + nozzleTemps slice-time override
- `data/SettingsBackupTest.kt` (16) — Export/import round-trip, version validation, partial restore, filament profile name resolution, stale skirt-loop import normalization
- `bambu/ThreeMfParserTest.kt` (12) - 3MF data model construction, isMultiPlate detection, hasPaintSupports field (B57)
- `bambu/BambuSanitizerTest.kt` (25) — INI config parsing, nil replacement, array normalization, filterModelToPlate, component size guard, group recentering
- `bambu/ProfileEmbedderTest.kt` (5) — convertToModelSettings: per-volume extruder preservation, remap, attribute order
- `bambu/LayerToolCustomGcodeXmlTest.kt` (2) — custom_gcode_per_layer.xml colour extraction for type 1 and 2 (parity with pause injector)
- `ui/ExtruderAssignmentTest.kt` (6) — ExtruderAssignment defaults, copy, list building
- `ui/FilamentJsonImportTest.kt` (15) — JSON import parsing: snake_case/camelCase, defaults, errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taylormadearmy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
