---
trigger: always_on
description: A personal iOS calendar app with an editorial / paper-stationery aesthetic. This file is the index: skim it first, then jump to the specific file(s) you need.
---

# Hibi — Agent Orientation

A personal iOS calendar app with an editorial / paper-stationery aesthetic. This file is the index: skim it first, then jump to the specific file(s) you need.

> **Before touching the calendar scrolling code** (`MonthsScrollView`, `StreamView`, `CalendarWindow`, `StreamWindow`), read [learnings.md](learnings.md) — it captures the SwiftUI infinite-scroll gotchas we hit and the patterns that survived.

## What it is

Hibi reads the user's system calendars via EventKit and presents them across three tabs in a single `NavigationStack`. The visual language is black-on-cream paper (dark mode is high-contrast black), Instrument Serif display type, hand-picked pastel tints. The Day tab uses a tear-off paper-pad metaphor (drag to rip, haptic on commit).

Not a backend-backed product. No account, no sync beyond the system calendar database. Weather is pulled from WeatherKit for the user's current location.

## Stack & targets

- **Platform:** iOS 26.0 minimum (`IPHONEOS_DEPLOYMENT_TARGET = 26.0`). Uses iOS 26 APIs (e.g. `Tab(..., value:)`, `MKReverseGeocodingRequest`, `ScrollPosition`).
- **Language:** Swift 5 toolchain, SwiftUI only. `@Observable` stores, `@MainActor` isolation, `nonisolated` delegate callbacks. No UIKit views — only `UIColor` for dynamic P3 colors in `PaperTints.swift`.
- **Frameworks:** EventKit, WeatherKit, CoreLocation, MapKit (reverse geocoding), CoreText (font registration).
- **Bundle id:** `com.weichart.hibi`. Entitlements: WeatherKit only.
- **No tests target.** No package dependencies. Pure Xcode project (`Hibi.xcodeproj`).

## Architecture

Two `@Observable` `@MainActor` stores own all non-view state. Views are passed them via `.environment(...)` from `ContentView`.

- **`EventStore`** — wraps `EKEventStore`. Caches events per `MonthKey` in `eventsByMonth`. Lazy-loads via `ensureLoaded(year:month:)` when a view scrolls into a new month. Watches `.EKEventStoreChanged` and reloads the months it already had. Owns the hidden-calendars set (persisted in `UserDefaults`) and a DEBUG-only demo mode that swaps in `DemoFixtures`.
- **`WeatherStore`** — `CLLocationManager` + `WeatherService`. 30-minute self-throttle. Keys daily forecasts by `(year, month, day)`. Also reverse-geocodes the location into a city name for the Day masthead.
- **`SampleData`** — `todayYear/Month/Day` are computed from `Date()` in the device's current time zone so views pick up the real "today." A separate `demoAnchorYear/Month/Day` (2026-04-18) is the fixed anchor for DEBUG demo screenshots; `isDemoAnchor(...)` is what triggers demo-mode time-of-day progress in `CalendarEvent.progress(...)`.

State flow for the three tabs lives on `ContentView`: `displayedYear/Month`, `selectedDay`, and a `scrollToNowToken` that views observe via `.onChange` to scroll back to today when the active tab is re-tapped.

## File index

### Entry + shell

- [HibiApp.swift](Hibi/HibiApp.swift) — `@main`. Registers the two Instrument Serif TTFs via CoreText at launch.
- [ContentView.swift](Hibi/ContentView.swift) — `TabView` with Month/Week/Day tabs, principal toolbar title, settings + add-event buttons, background radial gradient (cream in light, near-black in dark), appearance override wiring.

### Models / stores (`Hibi/Models/`)

- [CalendarEvent.swift](Hibi/Models/CalendarEvent.swift) — the view-layer event struct; includes `progress(at:)` with a demo-time-of-day branch. Also defines `WeatherCode` + `DayWeather`.
- [EventStore.swift](Hibi/Models/EventStore.swift) — see above. Pastelizes the EKCalendar color per event.
- [WeatherStore.swift](Hibi/Models/WeatherStore.swift) — see above.
- [SampleData.swift](Hibi/Models/SampleData.swift) — calendar math helpers, month/day name tables, `AppFont`/`AppColor` constants.
- [DemoFixtures.swift](Hibi/Models/DemoFixtures.swift) — hand-crafted events for screenshot days (DEBUG only).

### Views (`Hibi/Views/`)

- [MonthView.swift](Hibi/Views/MonthView.swift) — single-month grid. `MonthsScrollView` (same file) implements infinite vertical month scrolling; defines `MonthKey`.
- [StreamView.swift](Hibi/Views/StreamView.swift) — "Week" tab: infinite day-stream scroll with `ScrollPosition` + `StreamWindow` extension. Tapping a day jumps to the Day tab. Defines `DayKey` and `StreamWindow`.
- [DayView.swift](Hibi/Views/DayView.swift) — tear-off paper-pad day view. Three stacked paper cards with progressive tints (`PaperTints`), drag gesture rips the top sheet and commits prev/next day. Schedule list below.
- [SettingsView.swift](Hibi/Views/SettingsView.swift) — appearance picker, calendars link, DEBUG demo-mode toggle.
- [CalendarSelectionView.swift](Hibi/Views/CalendarSelectionView.swift) — hide/show individual EKCalendars (persists via `EventStore.setCalendar(_:hidden:)`).
- [PaperTints.swift](Hibi/Views/PaperTints.swift) — dynamic (light/dark) P3 paper-stack colors + `Color.pastelized(cgColor:)` for EKCalendar tints.

### Components (`Hibi/Views/Components/`)

- [EventCard.swift](Hibi/Views/Components/EventCard.swift) — stream-row event card.
- [DayEventRow.swift](Hibi/Views/Components/DayEventRow.swift) — row in the Day tab schedule.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexW00/hibi](https://github.com/AlexW00/hibi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
