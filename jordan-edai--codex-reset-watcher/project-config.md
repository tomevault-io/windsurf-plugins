---
trigger: always_on
description: This repo is **Codex Reset Watcher**, not Codex Cockpit.
---

# Agent Notes

This repo is **Codex Reset Watcher**, not Codex Cockpit.

Codex Reset Watcher is a small, open-source, local-first macOS menu bar app for
Codex usage limits and reset credits. Keep changes scoped to that product.

## Current State

- Public GitHub repo: `https://github.com/jordan-edai/codex-reset-watcher`
- Canonical local path: `/Users/everydayai/Documents/!Codex Projects/Rate Refresher Project`
- Compatibility path: `/Users/everydayai/Documents/Rate Refresher Project`
- Latest shipped release: `v0.4.4`
- Check `git log --oneline --decorate -5` for the current `main` commit; this
  note tracks the repo state through the `v0.4.4` weekly reset-day title fix.
- App bundle version is set in `script/build_and_run.sh`.

## Product Decisions

- The app must stay read-only.
- Do not redeem reset credits, reset usage, mutate account state, or add
  analytics.
- No OpenAI API key is required. The app uses the existing local Codex Desktop
  login at `~/.codex/auth.json`.
- The app currently calls internal Codex Desktop endpoints:
  - `https://chatgpt.com/backend-api/wham/usage`
  - `https://chatgpt.com/backend-api/wham/rate-limit-reset-credits`
- These endpoints can change without notice. Keep decoding tolerant and failure
  handling partial-data-friendly.
- The menu bar title currently shows weekly remaining capacity and the weekly
  reset weekday, for example `57% | Sunday`, with the status icon beside it.
  Never replace it with a banked-reset count, account label, or status sentence.
- There is temporarily no menu-title metric selector while Codex does not return
  the former 5-hour window. If weekly reset timing is missing, use `week`; if
  weekly data is unavailable, show `--% | week`. Do not use 5-hour,
  generic/unclassified, or reset-credit data as a current title fallback.
- Use an explicit SwiftUI `HStack` label for `MenuBarExtra`; a `Label` can
  collapse to icon-only in the real macOS menu bar even when the title string
  itself is correct.
- Display settings contain appearance only. Do not add a weekly/5-hour
  menu-title selector back until Codex reliably returns the 5-hour window. Read
  `MENU_BAR_DISPLAY_PLAN.md` before restoring it.
- Preserve the dormant 5-hour decoder, snapshot fields, and nudge logic. The
  5-hour window is expected to return, and its prior display design is recorded
  in `MENU_BAR_DISPLAY_PLAN.md`.
- Reset-credit rows in the dropdown should always label the date, for example
  `Reset 1 expires:`. The 5-hour and weekly usage rows should also label when
  those windows reset.
- Visual styling should flow through `CodexPalette` and `CodexStyle`. Prefer
  shared spacing, radius, type, row, and panel tokens over one-off view-local
  constants so the menu dropdown and desktop window stay visually aligned.
- Appearance mode is shared by the menu dropdown and desktop window. Keep
  Light/Dark/Auto routed through `CodexAppearanceMode`, SwiftUI
  `preferredColorScheme`, and `NSApp.appearance` so custom palette colors
  actually switch in menu-bar popovers.
- Usage capacity bars use remaining-percentage thresholds from the 2026 design
  refresh: green at 60% or higher, amber from 25% through 59%, and red below
  25%. Blocked usage windows override percentage color with danger styling.
- Keep desktop reset rows responsive and column-based. Labels/details and large
  expiry dates should never share one flexible inline text row, because that
  caused overlap at the default utility-window width.
- Keep the main desktop window roomy enough by default. The window min/default
  size tokens live in `CodexStyle.Size`, and `MainWindowController` expands
  restored undersized frames so older saved window sizes do not clip the footer.
- Routine app surfaces should stay light. Avoid smoky gray, dark tinted row
  fills, and dark terminal-block branding for normal states; use icons, borders,
  badges, and meters for emphasis, and reserve colored fills for warning/danger.
- Read `DESIGN_SYSTEM.md` before making visual changes.
- The menu dropdown uses fixed icon, content, metric, and date columns. Preserve
  that rhythm when adding rows so labels do not bleed into popover edges.
- Keep the menu dropdown comfortably sized. Do not solve menu fit by stacking
  overly dense rows; widen the popover and use readable row heights when reset
  credits and footer actions are visible together.
- The menu dropdown must hug its intrinsic content height. Do not add a
  screen-sized frame, forced viewport, `ScrollView`, or `ViewThatFits` wrapper
  that stretches it beyond the height its visible rows require.
- Menu section order is fixed: `Display settings`, `Current limits`, then
  `Banked Resets Expiration`, followed by the usage nudge.
- Cached snapshots and their cleanup actions belong only in the full desktop
  app. Do not add them back to the menu dropdown.
- The active account label can come from the usage response email, local
  `id_token` email/name, or the generic `Codex account` fallback. Do not expose
  account-ID suffixes in user-facing labels.
- Multi-account support is snapshot-based. The active account refreshes live;
  other accounts are cached last-seen snapshots only.
- Do not describe cached snapshots as simultaneous live accounts. They are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordan-edai/codex-reset-watcher](https://github.com/jordan-edai/codex-reset-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
