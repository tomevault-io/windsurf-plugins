---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Language Preference

**Always respond in Korean (한국어)** when interacting with this codebase. All explanations, summaries, documentation, and guidance should be written in Korean. Code identifiers, file/folder names, and technical terms should remain in English, but conceptual explanations and context should be in Korean.

## Project Overview

Compose-DateTimePicker is a Kotlin Multiplatform library providing date and time picker components for Compose. It targets Android, iOS, Desktop (JVM), and Web with a shared codebase, published to Maven Central as `io.github.kez-lab:compose-date-time-picker`.

**Repository VERSION_NAME**: 0.6.0
**License**: Apache 2.0

## Architecture

### Component Hierarchy

The library follows a **composition-based architecture** with a single generic `Picker<T>` component as the foundation:

```
Picker<T> (generic scrollable picker)
  ↓ composed into
TimePicker (hour + minute + optional AM/PM)
YearMonthPicker (year + month)
DatePicker (year + month + day)
```

**Key pattern**: Higher-level components (`TimePicker`, `YearMonthPicker`) are compositions of multiple controlled `Picker` instances, not subclasses. Generic `Picker<T>` receives `selectedItem` and `onSelectedItemChange` from the caller; higher-level state classes own only logical date/time values.

### Core Components

**`Picker.kt`** (~440 lines)
- Generic scrollable picker using `LazyColumn` with snap behavior
- Supports infinite (`isInfinity=true`) and bounded scrolling
- Uses a bounded cyclic buffer (`items.size * 1000`) for infinite mode
- Fading edge effect via `graphicsLayer` + `BlendMode.DstIn`
- State updates via `snapshotFlow { firstVisibleItemIndex }` in `LaunchedEffect`

**State files**
- `time/TimePickerState.kt` contains `TimePickerState` and `rememberTimePickerState`
- `date/DatePickerState.kt` contains `DatePickerState` and `rememberDatePickerState`
- `date/YearMonthPickerState.kt` contains `YearMonthPickerState` and `rememberYearMonthPickerState`
- `TimePickerState` exposes `selectedTime: LocalTime` and `selectedHourOfDay`
- `YearMonthPickerState` exposes `selectedYearMonth: YearMonth`; `selectedMonthDate: LocalDate` is for interoperability with date APIs
- Specialized picker states use saveable state; generic `Picker<T>` is controlled by caller-owned state because arbitrary `T` is not guaranteed saveable

**`DatePicker.kt`** / **`DatePickerState.kt`**
- Compose year, month, and day pickers
- Clamp selected day when the selected year/month has fewer days
- `DatePickerState` exposes `selectedDate: LocalDate`

**`TimePicker.kt`** / **`YearMonthPicker.kt`**
- Compose multiple `Picker` instances in a `Row`
- Handle 12/24-hour format conversion (TimePicker only)
- Use `kotlinx-datetime` for date/time utilities

### State Management Flow

```
User scrolls LazyColumn
  → LazyListState.firstVisibleItemIndex changes
  → snapshotFlow emits new index
  → Picker calls onSelectedItemChange(item)
  → caller-owned selectedItem updates
  → Recomposition shows updated UI
```

### Multiplatform Structure

```
datetimepicker/src/
├── commonMain/kotlin/    # Shared UI and logic
├── androidMain/          # Android-specific (UI tooling preview)
├── iosMain/              # iOS-specific (currently minimal)
├── desktopMain/          # Desktop-specific (currently minimal)
├── jsMain/               # Web-specific source set directory, currently minimal
└── commonTest/           # Shared unit tests
```

Most logic lives in `commonMain`. Platform-specific code is minimal.

## Development Commands

## Maintainer Workflow Preferences

- Use `feature/*` branch names for new implementation work in this repository.
- Treat `main` as the integration branch. There is currently no `develop` branch on the remote.
- After a substantial implementation step, run a six-agent feedback loop when the maintainer asks for autonomous improvement work: collect feedback, fix actionable issues, verify again, then open or update the PR.
- Merge PRs only after relevant local verification and GitHub Actions checks pass.
- Keep improving toward Android developer ergonomics first: state APIs, sample usability, documentation clarity, accessibility, and predictable behavior in real app lifecycles.
- Before adding custom performance machinery, verify that the standard Compose/runtime primitives are insufficient. Prefer `remember`, `derivedStateOf`, `snapshotFlow`, stable object ownership, and domain helpers before introducing bespoke caches or dirty-check classes. If the reason is performance, record the measured or directly inspected evidence in the PR.
- Treat bespoke caching, manual invalidation, synchronization, and equality-key logic as high-risk code. If such code remains, add focused regression coverage for stale data, source identity changes, non-default column orders, and state changes that should invalidate derived values. Private code still needs tests when it replaces framework guarantees.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kez-lab/Compose-DateTimePicker](https://github.com/kez-lab/Compose-DateTimePicker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
