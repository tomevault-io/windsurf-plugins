---
trigger: always_on
description: This file is the **source of truth** for conventions in this repository. It applies to any
---

# AGENTS.md — Working agreement for mectrics

This file is the **source of truth** for conventions in this repository. It applies to any
AI agent or human contributor. `CLAUDE.md` points here.

## 0. Golden rule: English-only repository

**Everything committed to this repo is in English** — no exceptions:

- Source code: identifiers, types, function names, variables.
- Comments and documentation comments.
- All Markdown docs (`README.md`, `docs/**`).
- Commit messages, branch names, PR titles/descriptions.
- User-facing UI strings: **English is the base/development language** (localized to other
  languages via the String Catalog — see §2).

The only place another language may appear is a live chat conversation with the user (who
may write in Turkish). Nothing from that chat leaks into the repo in another language.

## 1. Project shape

- `Packages/MetricsKit/` — UI-independent metric engine (SwiftPM). Providers, scheduler,
  ring-buffer store, engine. **No UI, no localization** (data-only, English identifiers).
  - `swift build`, `swift test`, `swift run mectrics-cli` (live terminal readout).
  - Builds in the **Swift 6 language mode** and must stay warning-free. `MetricProvider`
    requires `Sendable`; providers are `@unchecked Sendable` because the engine samples
    them on one serial queue. Guard anything read outside that queue with a lock.
- `Mectrics/` — the menu bar app (SwiftUI + AppKit).
- `project.yml` — XcodeGen project definition. **This is the source**; `Mectrics.xcodeproj`
  is generated. After editing `project.yml` **or adding/removing source files**, run
  `xcodegen generate`.
- `docs/` — the architecture deep dive, and nothing else. Contributor-facing only: planning
  notes, roadmaps, backlogs, and maintainer-only runbooks do not belong in the public
  repository. The release procedure lives in `scripts/release.sh`, which is self-documenting
  through its required environment variables.

Do **not** commit: `Mectrics.xcodeproj/`, `DerivedData/`, `.build/` (see `.gitignore`).

## 2. Internationalization (i18n)

- All user-facing strings go through `String(localized:)` or SwiftUI `Text`/`Label`.
- Never hardcode user-facing prose as a plain `String` without localization.
- App strings live in `Mectrics/Resources/Localizable.xcstrings`; widget strings live in
  `MectricsWidget/Localizable.xcstrings`. Both catalogs ship English, Turkish, Russian,
  Spanish, French, and Brazilian Portuguese.
- The General Settings language picker is backed by `AppLanguage`. Adding a language means
  adding its case and identifier there, then translating every entry in both catalogs.
- Module display names: use `MetricID.localizedName` (app layer), not the package's
  `displayName` (which is the English fallback).
- Numeric/symbolic menu-bar strings (percentages, rates, arrows) are not localized.

## 3. Menu bar rendering rules

- **Item width must be stable.** Each module reserves a fixed text width from a worst-case
  template (`MetricStatusItem.template(for:)`) and right-aligns text inside it. Item width
  must never depend on the current value's digit count — this prevents items from shifting.
- Use `NSFont.monospacedDigitSystemFont` so digits are equal width.
- If you add a module or change a format, update its template so real values never exceed
  the reserved width.
- **A module may contribute several items.** Components are independent toggles
  (`AppModel.toggleComponent(_:for:)`), so Battery can show icon + health at once.
- **Every component includes a readable value.** A chart-only item is not offered:
  a sparkline with no number cannot be read at a glance.
- **Absence is not zero.** When a reading is missing, render a dash and never fabricate
  `0%` / `0`. Do not offer a component whose data this Mac cannot report.
- Components are picked by clicking a live preview chip, not from a select box — the
  user chooses what they can see.

## 4. Surfaces and Settings

- **The menu bar is the only live surface.** The always-on-top floating panel and its
  global hotkey were removed; the optional **Compact Health** item is the supported
  overview. Do not reintroduce a second always-visible rendering surface.
- **Settings holds configuration, not routine actions.** Quit, copy, and export belong to
  the surfaces that own them (popover, Diagnostics, Attention Log), not to a preferences
  pane. The destructive, one-time app removal action is the sole exception because no
  other surface owns the app lifecycle.
- Every Settings pane uses `Form(.grouped)` and shares one window size — switching tabs
  moves the selection, never the window.
- Prefer progressive disclosure over dimmed controls: hide a control that cannot act yet
  and show its current value as text instead.

## 5. Performance & privacy invariants

- **Zero telemetry.** The only network calls allowed are (optional) update checks. No usage
  or hardware data ever leaves the device.
- Adaptive sampling: faster on AC, slower on battery; pause work that isn't visible —
  a sleeping display, a locked screen, and a switched-away session all count as invisible.
- Keep the hot path allocation-free (the ring buffer is pre-allocated).
- Targets: < 60 MB RAM, low/steady CPU, "Energy Impact: Low" in Activity Monitor.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farukkamcici/mectrics](https://github.com/farukkamcici/mectrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
