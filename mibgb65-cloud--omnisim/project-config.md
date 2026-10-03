---
trigger: always_on
description: This repository contains **OmniSIM**, a native Android SIM/eSIM renewal manager.
---

# OmniSIM repository guidance

This repository contains **OmniSIM**, a native Android SIM/eSIM renewal manager.
Treat this file as the local implementation contract for all work in this repository.

## Engineering behavior

- State material assumptions before implementing ambiguous behavior.
- Prefer the simplest solution that fully satisfies the requirement.
- Keep changes surgical: do not refactor or reformat unrelated code.
- Every changed line should trace to a product requirement or a test/build fix.
- Work in verifiable increments and keep the project compilable.
- Put business logic outside composables and cover critical logic with unit tests.
- Do not leave fake data paths, placeholder core actions, or core-feature TODOs.
- Keep every version-controlled Kotlin or Gradle Kotlin source file (`*.kt`, `*.kts`)
  at or below 600 physical lines, including comments and blank lines. Generated and
  build-output directories are excluded. Split files by responsibility instead of
  suppressing or weakening the `checkCodeFileLength` verification task.

## Product identity and scope

- App name: `OmniSIM`
- Subtitle: `SIM & eSIM Renewal Manager`
- Tagline: `Never miss a SIM renewal again.`
- Package: `app.omnisim.android`
- License: MIT
- Platform: Android only
- Product: lightweight, local-first renewal/recharge/keep-alive reminders for a single
  user managing a small number of SIMs and eSIMs.

Never add accounts, login, cloud sync/backend, billing, analytics, telemetry,
advertising, desktop/web/iOS targets, eSIM provisioning, carrier integration,
payments, SMS/call/contact access, scanning/OCR, AI, widgets, or multi-user features.

## Technical direction

Use native Android technologies:

- Kotlin and Jetpack Compose
- Material 3 and Navigation Compose
- Room/SQLite
- DataStore Preferences
- Coroutines and Flow
- WorkManager and Android notification APIs

Use a simple dependency flow:

```text
Compose UI -> ViewModel -> Repository -> Room / DataStore
```

A small application container is preferred over dependency-injection ceremony.
Use immutable UI state and `StateFlow`. Keep date, validation, reminder, backup,
and persistence logic out of composables. Avoid unnecessary third-party libraries.

## Core experience

The app must quickly answer:

1. What SIMs do I have?
2. Which SIM needs attention next?
3. After renewal, when is the next renewal?

Primary flow:

```text
Open -> see nearest renewal -> select SIM -> recharge externally
-> Mark as Renewed -> confirm actual date -> calculate/edit next date
-> save history -> reschedule reminders
```

Use exactly four bottom-navigation destinations: Home, SIMs, Usage, Settings. Renewal
history belongs on the SIM detail screen. Usage is a first-class destination and
must not be removed as unreachable or treated as dead code. The top app bar may
expose Add SIM.

## Design contract

- Polished native Android utility: minimal, calm, clean, modern, utility-first.
- Use Material 3, edge-to-edge layouts, proper system bars and accessible semantics.
- Support light, dark, system theme and optional Material You dynamic color.
- Avoid gradients, glassmorphism, neon, decorative charts, excessive shadows/cards,
  crowded layouts, and excessive animation.
- Use text as well as restrained color for status. Keep touch targets and contrast
  accessible and use stable keys in lazy lists.
- No onboarding carousel. Use polished empty states and concise snackbars.

## Navigation and screens

### Home

Home is not an analytics dashboard. Sort active SIMs by next renewal date. The
nearest record is visible first. Show urgent records under **Needs attention**
(Overdue, Due Today, Due Soon) and the rest under **Upcoming**. Urgent cards show
name/carrier, masked number, time remaining, exact date, cycle, and
**Mark as Renewed**. With no records, show an Add SIM empty state.

### SIM list

Show a clean vertical list with name, carrier/country where available, masked
phone number, next date, days remaining, and textual status. Provide search over
name, carrier, phone number, and country. Filters: Active, Due Soon, Overdue,
Archived. Archived records do not appear in Home.

### Usage

Show local renewal-cost summaries for active SIMs without turning Home into an
analytics dashboard. Present daily, 30-day, and 365-day estimates, per-currency
breakdowns, data-coverage guidance, and links to complete missing price or cycle
information. A combined total may use public European Central Bank reference
rates in the user's default currency. Cache the most recent valid rates for
offline fallback and clearly label loading, cached, partial, and unavailable
states. Do not add telemetry, tracking, decorative charts, or remote SIM-data
processing.

### Add/edit SIM

Use a full-screen Compose form. Required: display name, carrier, next renewal
date. Optional: phone, country, SIM type, plan, last renewal date, renewal cycle,
amount, currency, renewal website, notes. SIM types are `eSIM` (default) and
`Physical SIM`. Cycle options are 30, 60, 90, 120, 180, 365 days, Custom,
monthly on a fixed day from 1 through 31, and no automatic cycle. A monthly day
that does not exist in a shorter month resolves to that month's final day. Use
native Material date pickers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mibgb65-cloud/OmniSIM](https://github.com/mibgb65-cloud/OmniSIM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
