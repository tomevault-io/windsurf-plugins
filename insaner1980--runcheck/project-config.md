---
trigger: always_on
description: runcheck is a native Android app (Kotlin + Jetpack Compose) that monitors device health across four categories: battery, network, thermal, and storage. It provides real-time diagnostics, a unified health score, long-term trend tracking, and storage cleanup tools.
---

# CLAUDE.md — runcheck Project Instructions

## Project Overview

runcheck is a native Android app (Kotlin + Jetpack Compose) that monitors device health across four categories: battery, network, thermal, and storage. It provides real-time diagnostics, a unified health score, long-term trend tracking, and storage cleanup tools.

## Tech Stack

- **Language:** Kotlin 2.3.0 (via AGP 9.1.0 built-in Kotlin)
- **UI:** Jetpack Compose with Material 3 (BOM 2026.03.00), single dark theme
- **Min SDK:** 26 (Android 8.0)
- **Target SDK:** CinnamonBun beta (Android 17)
- **Compile SDK:** CinnamonBun beta (Android 17)
- **Architecture:** MVVM with Clean Architecture layers (data → domain → ui)
- **Database:** Room 2.8.4 for local historical data
- **Async:** Kotlin Coroutines 1.10.2 + Flow
- **DI:** Hilt 2.59.2
- **Charts:** Custom Compose components (TrendChart with axes/grid/tooltip/quality zones, AreaChart, HeatStrip, SegmentedBar)
- **Build:** Gradle 9.4.0 with Kotlin DSL, AGP 9.1.0, KSP 2.3.1
- **Localization:** English only (Finnish translations preserved in git history for future use)

## Project Structure

```
app/src/main/java/com/runcheck/
├── data/
│   ├── appusage/      # App battery usage data source
│   ├── battery/       # BatteryManager wrappers, BatteryDataSourceFactory,
│   │                  #   GenericBatterySource + manufacturer-specific sources
│   │                  #   (Android14, Samsung, OnePlus), BatteryCapacityReader
│   ├── billing/       # BillingManager (lifecycle-aware billing service),
│   │                  #   ProStatusCache
│   ├── charger/       # Charger comparison data
│   ├── device/        # Device detection, DeviceProfile, DeviceProfileProvider,
│   │                  #   DeviceCapabilityManager
│   ├── db/            # Room database (RuncheckDatabase with inner Converters, RoomTransactionRunner)
│   │   ├── dao/       # Room DAOs (Battery, Network, Thermal, Storage, Charger, etc.)
│   │   └── entity/    # Room entities (BatteryReadingEntity, etc.)
│   ├── export/        # Data export functionality
│   ├── insights/      # Persisted insights repository, debug seed helpers
│   ├── network/       # ConnectivityManager, TelephonyManager, SpeedTestService,
│   │                  #   LatencyMeasurer, NetworkRepositoryImpl, SpeedTestRepositoryImpl
│   ├── preferences/   # DataStore preferences
│   ├── storage/       # MediaStoreScanner, StorageCleanupHelper,
│   │                  #   CleanupPagingSource, StorageCleanupRepositoryImpl
│   └── thermal/       # ThermalManager, thermal/throttling repositories
├── domain/
│   ├── insights/      # Insight models (InsightModels.kt), rules, engine (+ ranking policy)
│   ├── model/         # Domain models (BatteryState, NetworkState, StorageState, etc.)
│   ├── usecase/       # Business logic (37 use cases)
│   ├── repository/    # Repository interfaces
│   └── scoring/       # Health score algorithm
├── ui/
│   ├── home/          # Single home screen (hub) + ViewModel
│   ├── insights/      # Dedicated Insights screen + ViewModel
│   ├── battery/       # Battery detail screen + ViewModel + session stats
│   │                  #   + BatteryInfoContent, BatteryInfoCards
│   ├── network/       # Network detail + SpeedTest screens + ViewModel
│   │                  #   + NetworkInfoContent, SpeedTestInfoContent, NetworkInfoCards
│   ├── thermal/       # Thermal detail screen + ViewModel + session min/max
│   │                  #   + ThermalInfoContent, ThermalInfoCards
│   ├── storage/       # Storage detail screen + ViewModel + cleanup/
│   │                  #   (CleanupScreen, CleanupViewModel, FileListItem, CategoryGroup)
│   │                  #   + StorageInfoContent, StorageInfoCards
│   ├── charger/       # Charger comparison screen + ViewModel
│   ├── appusage/      # App battery usage screen + ViewModel
│   ├── learn/         # Learn section — article catalog, list screen, detail screen
│   ├── settings/      # Settings screen + ViewModel
│   ├── pro/           # Pro upgrade screen, trial UI, purchase flow
│   ├── theme/         # Dark theme, color tokens, typography, spacing, motion tokens
│   ├── common/        # UiText, UiFormatters (formatPercent, formatTemp, etc.)
│   ├── chart/         # ChartHelpers (quality zones, axis labels, qualityZoneColorForValue),
│   │                  #   ChartModels, ChartRenderModel, ChartAccessibility
│   ├── fullscreen/    # FullscreenChartScreen + ViewModel (landscape-only)
│   ├── components/    # 34 shared composables (see Components below)
│   │   └── info/      # InfoSheetContent, InfoIcon, InfoBottomSheet, InfoCard, InfoCardCatalog, CrossLinkButton
│   └── navigation/    # NavGraph + Screen sealed class (push-based from Home)
├── pro/               # Pro/trial state management
├── billing/           # Billing state helpers
├── widget/            # Glance widgets (BatteryWidget, HealthWidget, WidgetDataProvider)
├── worker/            # WorkManager workers (Insights generation and related jobs)
├── service/
│   └── monitor/       # HealthMonitorWorker, HealthMaintenanceWorker,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Insaner1980) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
