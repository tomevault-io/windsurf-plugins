---
trigger: always_on
description: This is `@mobiscroll/javascript-lite` — the free, open-source (Apache-2.0) subset of Mobiscroll
---


# Mobiscroll for JavaScript — AI Rules (Lite)

This is `@mobiscroll/javascript-lite` — the free, open-source (Apache-2.0) subset of Mobiscroll
UI for plain/vanilla JavaScript. No framework required, no license key, no CLI:
`npm install @mobiscroll/javascript-lite`. Preact is bundled internally to render components but
is never part of the public API — components are used through plain DOM attributes.

**Not included in this package:** Eventcalendar, Scheduler, Timeline, Agenda, Calendar,
Datepicker, Range, Select. Those are commercial, live in `@mobiscroll/javascript` (not `-lite`),
and need a trial/license installed via the Mobiscroll CLI. If the user asks for scheduling, a
date picker, or a dropdown-with-search "Select", say so — do not try to build it from this
package's primitives, and do not import from `@mobiscroll/javascript`.

## Scope

USE this file when:

- Project has `@mobiscroll/javascript-lite` in package.json
- Code imports `from '@mobiscroll/javascript-lite'` or uses the global `mobiscroll` object
- No framework (React/Angular/Vue/jQuery) is in play

DO NOT use this file for React, Angular, Vue, or jQuery projects, and do not apply it to
`@mobiscroll/javascript` (the full/commercial package — different scope, same import shape).

## Component Mapping

| `mbsc-*` attribute                                                         | What it's for                                                                                                     |
| :------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `mbsc-button`                                                              | Buttons, icon buttons                                                                                             |
| `mbsc-input`, `mbsc-dropdown`, `mbsc-textarea`                             | Text field, native `<select>`, native `<textarea>` — same options, different tag                                  |
| `mbsc-checkbox`                                                            | Single checkbox                                                                                                   |
| `mbsc-radio`                                                               | Radio buttons (group by shared native `name`, like plain HTML radios — there is no separate group component here) |
| `mbsc-segmented`, `mbsc-segmented-group`                                   | iOS-style segmented control, grouped for shared value                                                             |
| `mbsc-stepper`                                                             | Numeric +/- stepper                                                                                               |
| `mbsc-switch`                                                              | Toggle switch                                                                                                     |
| `mbsc-page`                                                                | Page/layout wrapper (theming root)                                                                                |
| `mbsc-popup` (also `mobiscroll.popup(el, options)`)                        | Modal, anchored popover, bottom sheet, inline panel                                                               |
| `mobiscroll.toast()`, `.snackbar()`, `.alert()`, `.confirm()`, `.prompt()` | Notification popups — imperative functions, no markup needed                                                      |

There is **no `Icon` component** exported from this package (icons are internal rendering
details used by Button/Input, etc. — pass an icon name string to those options instead). Not a
component but always available once the CSS is loaded: `.mbsc-grid`/`.mbsc-row`/`.mbsc-col-*`
utility classes (bootstrap-style flex grid), shipped in `grid-layout.scss`.

## Rules

- Package: `@mobiscroll/javascript-lite` — never `@mobiscroll/javascript` for these components
- CSS (required once): `import '@mobiscroll/javascript-lite/dist/css/mobiscroll.min.css'`, or a
  `<link>` to the same file at `dist/css/mobiscroll.min.css`
- Components initialise from the `mbsc-*` attribute in markup present at load — there is no
  per-element setup call needed for those
- For markup inserted into the DOM after initial load, call `mobiscroll.enhance(element)` to
  activate any `mbsc-*` attributes inside it
- Notification functions (`toast`, `snackbar`, `alert`, `confirm`, `prompt`) return Promises;
  `confirm` resolves to a boolean, `prompt` resolves to the entered string or `null`
- Types are prefixed `Mbsc` (e.g. `MbscButtonOptions`) and exported from the same package for
  TypeScript users
- `color` is one of `'primary' | 'secondary' | 'success' | 'danger' | 'warning' | 'info' | 'dark'
| 'light'` across button/checkbox/radio/segmented/stepper/switch

## Usage

```html
<input mbsc-input data-label="Email" type="email" id="email" />
<select mbsc-dropdown data-label="Country" id="country">
  <option value="us">United States</option>
</select>
<input mbsc-checkbox type="checkbox" data-label="Subscribe" id="subscribe" />

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acidb/mobiscroll](https://github.com/acidb/mobiscroll) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
