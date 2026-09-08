---
trigger: always_on
description: Off Work Countdown is one product with three delivery targets:
---

# Off Work Countdown Agent Guide

## Project overview

Off Work Countdown is one product with three delivery targets:

- Web: Next.js 15 App Router, React 19, TypeScript, Tailwind CSS and Serwist.
- Desktop: Tauri v2 using the same exported frontend plus a small Rust/AppKit shell.
- iOS: a native SwiftUI app in `src-mobile/ios`, with a WidgetKit extension.
  It does **not** embed a WebView and does not render any Next.js page. The
  Capacitor shell it replaced is gone; anything still mentioning `cap`,
  `CapApp-SPM` or `capacitor.config` is stale.

The three targets share business rules and translations, never markup. Web and
Desktop share the React tree; iOS reimplements the surface natively and
consumes the same rules through a generated bundle (see below).

The product is local-first. Work hours, salary and preferences stay on the
user's device by default. The user-approved 2026-09-05 expansion permits these
preferences and career salary history in opt-in private CloudKit sync and
user-triggered backup exports (plan 015). Do not add a product account system,
or place salary values in widgets, URLs, analytics payloads or share metadata.

## Important architecture boundaries

- `lib/countdown.ts` is the source of truth for shift calculations. Rust only
  keeps an absolute running snapshot alive when the WebView is hidden; do not
  create a second implementation of schedule rules in Rust.
- `lib/reminders.ts` is the source of truth for reminder timing and copy. It
  turns a shift into absolute trigger times; Rust only compares them against
  the clock. Do not move milestone, lunch-boundary or micro-break derivation
  back into Rust — that is what the 3.1.6 refactor removed. iOS schedules the
  same list up front, because a phone cannot poll every second.
  `lib/reminders.test.ts` is the acceptance spec for every consumer.
- iOS reaches those rules through `src-mobile/ios/App/App/Resources/CountdownRules.js`,
  generated from `lib/countdown.ts`, `lib/reminders.ts` and `lib/summary.ts` by
  `npm run build:ios-native-rules` and evaluated in JavaScriptCore. Swift only
  feeds it inputs and renders what comes back. **Do not port a rule into Swift**
  — a schedule, summary or salary calculation written twice is two answers, and
  the "This week" row has already shipped disagreeing values that way. If Swift
  needs a value the bundle does not expose, extend the bundle. A port is not
  forbidden forever, but it is gated: `plans/002-records-life-focus.md` sets the
  bar as a measurement, not an expectation, and any port must arrive with the
  TS-generated differential fixtures and an update to this rule in the same
  change. Until that measurement exists, extend the bundle.
- The generated bundle is a build artifact, not source. It is regenerated from
  the TypeScript, so never hand-edit it, and never let a Swift change depend on
  a bundle that was not rebuilt from the current `lib/`.
- Since 3.1, a running shift is `segments + plannedEndAtMs + overtimeEndAtMs`.
  Remaining time, progress and earnings must use effective segment duration;
  never reintroduce `end - now` or a standalone start/end range. Rust may only
  compare and sum the absolute segments prepared by the frontend.
- Web and Desktop are separate build targets selected by `BUILD_TARGET`.
  `npm run build` must preserve middleware and Route Handlers; `npm run
  build:desktop` must produce a static export in `out/` without Web-only APIs.
- Keep standard Next.js Route Handler filenames such as `route.ts`. Vercel's
  output tracing relies on them. Desktop exclusion belongs in the build target
  configuration, not in renamed route files.
- macOS Mini Timer is native AppKit in
  `src-tauri/native-mini/NativeMiniTimer.m`, linked by `src-tauri/build.rs`.
  macOS 26 uses `NSGlassEffectView`; older macOS uses Vibrancy. Do not replace
  it with a WebView or CSS glass effect.
- macOS 3.1 also has an optional WebView floating timer for the standard and
  woodfish skins. It is a separate window from the native menu-bar panel; do
  not merge their window lifecycle or make either one appear automatically in
  release builds.
- Windows uses the lightweight `/[lang]/mini` Desktop page and programmatic
  Tauri window creation. Platform-specific implementations are intentional.

## UI rules

These first six are the default for every target. The platform-specific rules
below them are exceptions that were argued for, not licence to restyle.

- **Default to restraint.** New UI should first look unremarkable for its
  platform, then earn any emphasis it gets. Visual weight is paid for by
  everything else on the screen, so nothing receives it without a reason.
- **Sizing and spacing are judgements, not a spec.** Decide them from the
  surface type, the information density, the platform convention, how often the
  control is used and where it sits in the hierarchy. Do not carry one set of
  numbers across surfaces, and do not enlarge something because it is
  important.
- **Secondary entry points stay out of the visual centre.** Settings, toggles,
  tool buttons, legends, info buttons, expand/collapse and source labels are
  support. They must not compete with the countdown, the calendar or a screen's
  main conclusion.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ififi2017/Off-Work-Countdown](https://github.com/ififi2017/Off-Work-Countdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
