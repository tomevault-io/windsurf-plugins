---
trigger: always_on
description: A cross-platform ballistic calculator built on **Avalonia UI**. It began as a rewrite of an older
---

# BallisticCalculator2 - Project Guide for Claude

## Project Goal

A cross-platform ballistic calculator built on **Avalonia UI**. It began as a rewrite of an older
WinForms application; that application is archived and is **not** a reference for this work any more.

## Development Workflow

**Trunk-based development.** Commit directly to `main`; do **not** create feature branches unless explicitly requested. Only commit/push when asked.

## Key References

### Libraries
- **Gehtsoft.Measurements**: Measurement units library located at `/mnt/d/develop/components/BusinessSpecificComponents/Gehtsoft.Measurements/Gehtsoft.Measurements/`
  - Provides generic `Measurement<T>` struct where T is a unit enum (DistanceUnit, VelocityUnit, WeightUnit, etc.)
  - Static method: `Measurement<T>.GetUnitNames()` returns `Tuple<T, string>[]` for populating unit lists

- **BallisticCalculator** (NuGet **1.1.13**; the `PackageReference`s pin exact versions, they are not ranges):
  Core ballistic calculation library.
  Source at `/mnt/d/develop/components/BusinessSpecificComponents/BallisticCalculator.Net/`.
  - Provides `BallisticCoefficient` struct, `DragTableId` enum, and the calculation engine.
  - **`.drg` metadata (1.1.11.2):** the header carries name, weight, diameter, **bullet length** and
    **source**; `DrgDragTable.Save`/`Open` round-trip all of them, and
    `RadarDragTableFactory.Create` takes optional `bulletLength`/`source`. Files written earlier store
    those two slots as `0` — treat non-positive as absent.
  - **Multi-BC scale (1.1.11.3):** `DrgDragTableFactory.Build` now returns the projectile's own Cd
    (`Cd_base(M)/BC(M) * SD`), the same scale a `.drg` stores, so a built table survives Save/Open and runs
    with the **form factor of 1** the factory stamps into the entry. Bullet **weight and diameter are
    required** (they set the scale) and the supplied BC is overwritten. Before this, a built table needed a
    BC *value* of 1.0 and was 1/SD (≈2.8×) too draggy once saved.
  - **Zeroing API (1.1.11):** `SightAngle` was removed — compute the zero with
    `TrajectoryCalculator.CalculateZeroParameters(...)` then `ShotParameters.Apply(zero)`. `ShotParameters`
    also has `ShotDropAdjustment`/`ShotWindageAdjustment` (dialed clicks), `BarrelAzimuth`, `Latitude`.
  - **`BallisticCalculator.Tools`** namespace: `PointBlankRange`, `MovingTargetLead`, `HitProbability`,
    `RadarDragTableFactory`, `BallisticCoefficientConverter` (see the `ballistic-calculator` skill).
  - **Atmosphere (1.1.12):** `Density` is now computed from the resolved station `Pressure` rather than the
    constructor's `pressure` argument — it was up to 21% high at 5000 ft with `pressureAtSeaLevel: true`. **No
    trajectory, zero or drag result changes** (the engine always read the resolved pressure); only the public
    property. `CreateICAOAtmosphere` no longer feeds `humidity` into the base-altitude slot (~0.008% of
    pressure, and only for calls that passed a humidity). New `Atmosphere.DensityAltitude` — the
    standard-atmosphere altitude matching this air's density; note its baseline is the ICAO sea-level density,
    **not** `Atmosphere.StandardDensity` (they differ ~0.005%, about 1.9 ft, and are not interchangeable).
  - **Named failures (1.1.13):** the engine now raises its own exception types instead of a bare
    `InvalidOperationException`, so the UI can tell a bad input from a bug:
    `ZeroRangeCantBeReachedException` (the load cannot reach the zero distance) and
    `TrajectoryCannotBeCalculatedException` (the numbers do not integrate — a zero BC, weight or muzzle
    velocity). Both derive from `InvalidOperationException`, so catch them **before** any broader handler.
    Mapped to user-facing sentences in `ShotCalculator.Explain`; everything else keeps its stack trace.
  - **`MilDotReticle`** is built in **milliradians** (12 mrad across, zero at 6 mrad, marks on whole mrad) —
    note `AngularUnit.Mil` is the military mil, 1/6400 of a circle, and ~1.9 % off a milliradian.
  - Consult the `ballistic-calculator` skill before using this API; it describes the **1.1.13+** surface,
    including both named exceptions and `MilDotReticle` in milliradians.

## Project Structure

```
BallisticCalculator2/
├── Common/
│   ├── BallisticCalculator.Types/            # Shared models + domain helpers (no UI)
│   │   ├── ShotData, ZeroingData             # data models
│   │   ├── ZeroingCalculator                 # ShotData -> library zeroing inputs / zero
│   │   └── ShotTrajectoryCalculator          # single source of truth: ShotData -> trajectory
│   ├── BallisticCalculator.Controls/         # Shared UI controls
│   │   ├── Controls/                         # MeasurementControl, ReticleCanvasControl, AzimuthDirectionControl, ...
│   │   ├── Controllers/                      # Pure logic (MeasurementController, SummaryController, ReticleOverlayController, ...)
│   │   ├── Canvas/                           # SkiaReticleCanvas
│   │   └── Models/                           # UnitItem, DragTableInfo, WindArrow
│   ├── BallisticCalculator.Panels/           # Input/output panels (RiflePanel, ParametersPanel, SummaryPanel, ...)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikolaygekht/ballistic.calculator.app.avalonia](https://github.com/nikolaygekht/ballistic.calculator.app.avalonia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
