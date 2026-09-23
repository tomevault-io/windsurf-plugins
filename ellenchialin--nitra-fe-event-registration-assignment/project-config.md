---
trigger: always_on
description: Event Registration Wizard — Vue 3.5.17 + Quasar 2.18.5 + UnoCSS. Interview assignment.
---

# CLAUDE.md

Event Registration Wizard — Vue 3.5.17 + Quasar 2.18.5 + UnoCSS. Interview assignment.
Narrative rationale lives in `PLAN.md`; this file is operational rules only.

**Where the work stands: `PLAN.md` §1, phase table and progress log.** Update it when a phase
completes — status and the delivering commits, not elapsed hours. Deliberately not mirrored here;
duplicated progress goes stale on one side and then misleads.

## Commands

```bash
nvm use 22.17.0   # package.json pins this; do not develop on another major
yarn dev          # http://localhost:9001 (note: not 9000, see quasar.config.js)
yarn build
```

Run `yarn lint && yarn format` before each commit.

## Source precedence

When sources conflict: **official assignment doc > repo README > Figma mockup**.
The Figma file is internally inconsistent in places — see `PLAN.md` §3 before treating a frame as
specification.

## Hard rules

- **No hardcoded hex.** Use UnoCSS semantic shortcuts (`text-neutral`, `bg-surface-l1`,
  `border-danger-emphasis`). Full list in `src/unocss/semantic.js`. This is an explicit grading
  criterion.
- **No arbitrary radius values.** Use the theme radius scale (2 / 6 / 10 / 12 / full). Do not write
  `rounded-[10px]`.
- **All date handling is UTC.** Group, compare, and display with
  `Intl.DateTimeFormat(..., { timeZone: 'UTC' })`. Never `getDate()`/`getHours()` — mock timestamps
  are `Z` and local-time conversion shifts `ws2` across a day boundary.
- **Money is integer cents.** Format only at the display boundary via `utils/currency.js`.
- **Derived state is `computed`, never `watch`.** There is exactly **one watcher**: the
  `watchEffect` in `src/boot/i18n.js` syncing `<html lang>` to the active locale — a DOM attribute,
  so genuinely external state, and annotated in place. Any further watcher must clear the same bar.
  The other candidate (URL step sync) was cut because that state was derivable.
- **No user-facing string literals.** All copy goes through `vue-i18n` keys (`en`, `zh-TW`).
- **JSDoc on every exported function** — params, return, and non-obvious behaviour.
- Render all event copy from `src/mocks/event.js`. Never hardcode the event name (the design's
  "WebDev Summit 2025" / "TechConf 2025" strings are stale; data says 2028). The static
  `<title>` in `index.html` is the deliberate exception — document metadata, not rendered copy,
  and deriving it in JS would only trade duplication for a title flash. If a title ever needs to
  vary by step or locale, use Quasar's Meta plugin rather than assigning `document.title`.

## Gotchas

- **The primary CTA is orange, not brand teal.** Use `bg-accent-emphasis-rest` (`#FB7429`).
  `--q-primary` in `src/css/colors.scss` is bound to brand teal, so Quasar's `color="primary"`
  renders the wrong button.
- **Use `text-accent-default` / `text-info-default` / `text-warning-default`, never the bare
  `text-accent` / `text-info` / `text-warning`.** The `-default` forms are the design system's
  canonical names — Figma's token reference frame lists `text/warning/default` and
  `text/info/default`, matching the CSS variables. The bare names are shorthand, and Quasar
  defines those three itself with `!important`, so the shorthand silently renders Quasar's
  palette (`text-warning` gives `#FADD00`, not the design's `#918108`). These are the only three
  colour collisions: every other semantic shortcut carries a suffix Quasar does not define, and its
  palette stops at `-14`.
- **The typography shortcuts collide too, but resolve our way.** Quasar also defines `.text-h1`
  through `.text-h6`, `.text-subtitle1` and `.text-subtitle2`. Unlike the three colour classes it
  declares these _without_ `!important`, and the UnoCSS sheet loads later, so ours win on cascade
  order alone — verified: `text-subtitle1` renders 16/20 at weight 610, not Quasar's 16/28 at 400.
  Nothing to fix, but do not reorder the stylesheets, and be aware a typography class built
  dynamically (which UnoCSS cannot scan) would fall through to Quasar's much larger scale.
- **Translate Figma tokens by value, never by name.** The brand ramp is shifted one step between
  the two: Figma's `bg/brand/muted/rest` is `#EEF6F7`, but the starter's `--bg-brand-muted-rest`
  is `#CBE5E6` — `#EEF6F7` lives in `bg-brand-subtle-rest`. `text/brand/default` and
  `text/brand/emphasis` disagree the same way. Copying the name out of Figma's generated CSS
  silently produces the wrong colour; look the hex up in `src/css/colors.scss` and use whichever
  token holds it. When no semantic token holds the value, use the **palette utility** for that hex
  (`text-teal-500` for `#3A7679`, `text-teal-700` for `#264D4F`, `text-orange-700` for `#A13B02`) —
  still theme-backed, and honest about the value. Do not snap to the nearest semantic token.
- **Inter Variable must be loaded.** The starter ships no webfont; the design's weight tokens
  (630/610/570/485) are meaningless without it.
- **Never use `hidden`.** Quasar declares `.hidden { display: none !important }`, which beats any
  responsive utility, so `hidden tablet:inline` stays invisible at every width. Use `lt-tablet:hidden`
  and friends — a class name Quasar does not define. Audited: of Quasar's 625 `!important` utility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ellenchialin/nitra-fe-event-registration-assignment](https://github.com/ellenchialin/nitra-fe-event-registration-assignment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
