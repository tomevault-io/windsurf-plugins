---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UgCS GeoHammer is a Java 21 desktop application for geophysical data processing and visualization. Built with Spring Boot 3.2 and JavaFX. Maven-based build (`pom.xml`). Package root: `com.ugcs.geohammer`.

### Build & Test Commands

```bash
mvn clean package                    # Full build (compiles with Error Prone + NullAway)
mvn test                             # Run all tests
mvn -pl . -Dtest=CsvParserTest test  # Run a single test class
mvn -pl . -Dtest=CsvParserTest#testParseLine test  # Run a single test method
mvn javafx:run                       # Run the application
mvn checkstyle:checkstyle            # Run checkstyle (warnings only, does not fail build)
```

Error Prone with NullAway runs during main compilation only (not test compilation). NullAway is configured in WARN mode with `OnlyNullMarked=true` and JSpecify annotations.

### Key Packages

- `chart/` — Chart rendering (GPR, sensor line charts, tool views)
- `format/` — File format parsers (CSV, DZT, SGY, SVLOG)
- `map/` — Map rendering layers (GPS track, grid, satellite, radar)
- `model/` — Domain model and events
- `service/` — Business logic services (gridding, palette, GPR processing)
- `view/` — Reusable JavaFX UI components
- `util/` — Utility classes
- `math/` — Mathematical algorithms (interpolation, filtering, smoothing)

---

## Architecture Overview

Entry point: `MainGeoHammer` (`MainGeoHammer.java`, extends `javafx.application.Application`).

### Bootstrap Sequence

`MainGeoHammer.init()` creates `AnnotationConfigApplicationContext("com.ugcs")` which scans 60+ Spring beans. The main Stage is built from `SceneContent` (`SceneContent.java`), which assembles a three-panel layout: `MapView` (`map/MapView.java`, left) | `ProfileView` (`ProfileView.java`, center) | `OptionPane` (`chart/tool/OptionPane.java`, right), with `StatusBar` (`StatusBar.java`) at the bottom. `AppContext` (`AppContext.java`) provides static access to the Spring context and primary Stage.

### Event-Driven Coordination

Components communicate via Spring's `ApplicationEventPublisher`. All events extend `BaseEvent` (`model/event/BaseEvent.java`). Publish via `model.publishEvent(new SomeEvent(...))`. Listen via `@EventListener` on `private void` methods.

#### Event Catalog

**File lifecycle:**
- `FileOpenedEvent` — file(s) successfully loaded. Published by `Loader`. Listened by `ProfileView`, `GpsTrack`, `SatelliteMap`, `TraceCutter`, etc.
- `FileSelectedEvent` — user selects a file to view. Published by `Model`/charts. Listened by 20+ components (tools, layers, views).
- `FileClosedEvent` — file removed. Published by `Chart.close()`. Listened by `Model`, `GpsTrack`, `GridLayer`, `UndoModel`, etc.
- `FileUpdatedEvent` — file data modified (e.g., after cropping). Published by `Model.cropTraces()`, `SensorLineChart`.
- `FileRenameEvent` — file renamed. Published by `Saver`. Listened by `GridLayer` (invalidates cached grids).
- `FileOpenErrorEvent` — file opening failed. Published by `Loader` on exceptions.

**Data/series:**
- `SeriesSelectedEvent` — data series selected in `SensorLineChart`. Listened by `GriddingTool`, `ScriptExecutionTool`.
- `SeriesUpdatedEvent` — series visibility/selection changed. Listened by `GriddingTool`.
- `GridUpdatedEvent` — gridding (spatial interpolation) completed. Published by `GridLayer`. Listened by `PaletteView`.
- `DepthRangeUpdatedEvent` — GPR depth slider moved. Listened by `Model` (syncs depth across all open GPR files).
- `TemplateUnitChangedEvent` — data units changed (e.g., ns to m). Listened by `Model` (updates depth sliders).

**Task lifecycle:**
- `TaskRegisteredEvent` / `TaskCompletedEvent` — async task started/finished. Published by `TaskService`. Listened by `TaskStatusView`.

**State:**
- `UndoStackChanged` — undo stack modified. Published by `UndoModel`. Used to enable/disable undo button.
- `WhatChanged` — generic UI/rendering state change with `Change` enum: `traceValues`, `traceCut`, `windowresized`, `justdraw`, `mapscroll`, `profilescroll`, `mapzoom`, `adjusting`, `updateButtons`, `fileSelected`, `csvDataZoom`, `traceSelected`. Listened by 15+ components for rendering updates.

#### Key Event Workflows

**File open:** `Loader.load()` → `FileOpenedEvent` → `ProfileView`, `GpsTrack`, `SatelliteMap` update → `WhatChanged(updateButtons, justdraw)` triggers rendering.

**File select:** Chart/Model → `FileSelectedEvent` → 20+ components load state for selected file (tools, layers, palette, inspector).

**Data modification (crop/filter):** `Model.cropTraces()` → `FileUpdatedEvent` → `WhatChanged(traceCut)` → `GridLayer` invalidates grid, `GpsTrack`/`QualityLayer` refresh, charts redraw.

**Gridding:** `GridLayer.performGridding()` async → `GridUpdatedEvent` → `PaletteView` refreshes.

**Depth sync:** Depth slider moved → `DepthRangeUpdatedEvent` → `Model` syncs across all open GPR files.

**Task tracking:** `TaskService.registerTask()` → `TaskRegisteredEvent` → `TaskStatusView` shows progress → task completes → `TaskCompletedEvent` → status bar clears.

### Central State: Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ugcs/GeoHammer](https://github.com/ugcs/GeoHammer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
