---
trigger: always_on
description: Guidance for working in this repo. Keep it short; update it when the architecture changes.
---

# CLAUDE.md

Guidance for working in this repo. Keep it short; update it when the architecture changes.

## What this is

A small **Manifest V3 browser extension** (Brave/Chrome/Chromium) that makes
web apps follow the current [Omarchy](https://omarchy.org/) theme. One
app-agnostic **engine** + one **pack per site**: Slack (`content.js`, the full
pack — repaints chrome/sidebar/message pane and auto-flips Slack's Light/Dark
Color Mode), WhatsApp Web (`whatsapp.js`, declarative), GitHub (`github.js`,
declarative Primer tokens), Linear (`linear.js`), Discord (`discord.js`),
Outlook Web (`outlook.js`, Fluent v9 tokens), Notion (`notion.js`,
`--c-`/`--ca-` tokens + a Prism syntax palette), and HEY email + calendar
(`hey.js`, one pack for both). A **bash
native-messaging host** reads the active Omarchy theme from
`~/.local/state/omarchy/current/` and pushes theme changes to the extension
the moment they land. **Requires Omarchy 4+.**

This is end-user desktop tooling, not a web service. There is no build step, no
package manager, and no test suite — it's plain JS plus a dependency-free bash
script.

## Layout

- `extension/` — the unpacked extension
  - `omarchy-colors.js` / `omarchy-surfaces.js` / `omarchy-runtime.js` — the
    app-agnostic engine (listed first in each pack's own content-script entry,
    so it loads before the pack and shares its scope): color helpers
    (`relLuminance` linearizes channels per WCAG, `contrastRatio`,
    `alphaForContrast`, `inkOn` for ink that rides on a saturated fill,
    `toTriplet` for triplet-valued design tokens),
    `deriveSurfaces()` (the theme→surfaces contract), and the `OmarchyTheme`
    registry that receives themes and dispatches to the registered pack.
    `sidebarMuted` is **contrast-targeted, not a fixed fraction of `fg`**: its
    alpha is solved per theme so muted copy clears 6:1 against the page bg
    (matching GitHub's own dark muted at 6.15:1). A flat 0.65 put 5 of the 22
    shipped themes below the AA 4.5:1 floor — rose-pine worst at 3.00:1 — and
    packs spend this level on real copy, not incidental labels. It stays an
    `rgba()` ink so it keeps adapting to the surface it lands on.
  - `content.js` — **the Slack pack**, and the bulk of the work. Builds the
    themed CSS from the derived surfaces and injects it; drives Slack's
    Preferences modal to flip Color Mode; registers via `OmarchyTheme.register()`.
    See "content.js structure" below.
  - `whatsapp.js` — **the WhatsApp pack** (declarative tier): just a `cssVars`
    table mapping WhatsApp's own CSS custom properties to the derived surfaces.
    Every name verified against the live WhatsApp DOM (2026-08-04); the chat
    list/header paint via the `--WDS-surface-*` semantic tokens, found by
    resolving every custom property at the element and matching the painted
    color. Requires WhatsApp's theme set to "System default".
  - `github.js` — **the GitHub pack** (mostly declarative): maps Primer semantic
    tokens (`--bgColor-*`, `--fgColor-*`, `--borderColor-*`, `--header-*`,
    `--control-*`, `--overlay-*`) to derived surfaces. Verified via Playwright
    against public github.com (2026-08-04). `onColorMode` pins
    `data-color-mode`; Appearance → "Sync with system" is the supported setup.
    Matches `github.com` and `gist.github.com`.
    Also maps the `--brand-color-*` family: **signed-out GitHub is a second
    design system**, and its header/mega-menu read
    `var(--brand-color-canvas-default, var(--bgColor-default))` — since GitHub
    defines the brand token, the Primer fallback never applies. Only the ~37
    primitives are mapped; the ~560 per-component `--brand-<Component>-*` tokens
    are build-time literals, not references to them.
    **Skips GitHub's marketing site** (`/features/*`, `/pricing`, `/resources/*`,
    `/open-source`, the signed-out homepage, …): those pages deliberately mix
    dark and light heroes, so one omarchy surface flattens them. Detected by
    GitHub's own routing — `<meta name="route-controller">` starting with
    `site_` — never a URL prefix list, which would rot as pages are added.
    Opting out is simply *not registering*; the engine holds and replays the
    theme while no pack exists, so such pages get zero side effects. The pack
    runs at `document_start`, so it waits on a MutationObserver for that meta
    rather than `DOMContentLoaded`, which would flash unthemed app pages.
    Its small `apply` hook exists for one thing: GitHub fills the header search
    icon from `--fgColor-onEmphasis`, which correctly resolves to the page
    background on dark themes — right for text on an accent fill, invisible
    here — so that selector is overridden instead of the token retuned.
  - `linear.js` — **the Linear pack**: semantic tokens (`--bg-*`, `--color-*`,
    `--focus-*`) plus a dynamic remap of Linear's hashed StyleX `--sx-*` slots,
    each classified by USAGE (which CSS property consumes it — background →
    surface, color/fill → text; grey-level guessing is under-determined and
    broke light mode) and re-stomped via a fast rAF paint path on re-renders.
    Three measurements (2026-09-22, live board + issue views) shape the scan:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scottjones/omarchy-webapp-theme](https://github.com/scottjones/omarchy-webapp-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
