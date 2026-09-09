---
trigger: always_on
description: Cloudflare Worker that renders a Last.fm "recently played" card as an SVG for GitHub profile READMEs, plus a static configurator that builds the embed snippet.
---

# lastfm-recently-played

Cloudflare Worker that renders a Last.fm "recently played" card as an SVG for GitHub profile READMEs, plus a static configurator that builds the embed snippet.

## Commands

```bash
npm run dev          # wrangler dev on :8787
npm run typecheck
npm test
npm run deploy
npx wrangler secret put LASTFM_API_KEY   # production key; never in wrangler.jsonc
```

Pushing to `main` runs typecheck + tests in GitHub Actions; Cloudflare Workers Builds deploys.

## Conventions

American spellings throughout - code, comments, docs and UI. The URL parameters
are `bg_color`, `text_color` and so on, and prose that says "color" beside them
reads as a different thing.

## What the rendering context forces

GitHub renders the card inside an `<img>`, proxied by Camo. Almost every design decision follows from that:

- **The SVG must be self-contained.** No external fonts, images, CSS or JS - none of it loads. Album art is fetched server-side and inlined as a base64 data URI.
- **Links and `:hover` are inert there.** They work when the URL is opened directly, so keep them, but never make anything depend on them.
- **Errors return HTTP 200 with a valid SVG.** A 4xx renders as a broken-image icon with no explanation and poisons Camo's cache with the failure.
- **Everything must finish inside Camo's ~10s socket timeout.** All upstream work shares one `Deadline` (`src/util/deadline.ts`). Don't add independent timeouts - they sum.

## Option model

`profile` (`header` / `footer-left` / `footer-right` / `off`) says **where** your identity goes; `username` and `avatar` say **what's in it**. Folding placement into content is what once made the `avatar` toggle look header-only. The footer holds exactly **one** thing - when `profile` is in the footer, `footer` is ignored rather than stacked underneath.

## Configurator

`public/` is plain HTML, CSS and JS with no build step, and the page's whole job is to assemble a URL string.

- **Only values that differ from the theme reach the URL.** The color fields are filled in with whatever the card is using, so they can be copied out, and a field equal to its theme value is treated as unset. Resetting writes the theme's value back.
- **Color swatches are bound to `change`, not `input`.** A native picker fires continuously while dragging and every preview is a request to the Worker.
- `THEME_COLORS` mirrors each theme's settable colors from `render/themes.ts`. Update both together, or an untouched picker shows a color the card isn't using.
- The form is a fixed 500px and the preview takes the remaining width, since the card can run up to 1000px wide and the form gains nothing from more. Card height changes with track count, but the preview is in its own column, so it never reflows the form.
- **The snippet sits above the preview, at the top of the right column, and the whole column is sticky.** It's what the page exists to produce, so it stays in view while options are worked through - lower down, it scrolled off screen at the moment it mattered. The image URL has no box of its own; `Copy URL` reads it from state, since a second code box would just repeat a string already on screen.

## Gotchas

- **Every string from Last.fm goes through `escapeXml`.** One bare `&` breaks the entire image, silently, with nothing in any log. This is the most common way to ship a broken card.
- **Caller-supplied colors are validated, not escaped.** Every color parameter is interpolated into an SVG attribute, so `parseHexColor` allowlists a strict hex shape and returns null otherwise. Any new color parameter must go through it - this is a security boundary, not a formatting preference.
- **Most of `Theme` is derived, not chosen.** `bg_color`, `text_color`, `artist_color`, `meta_color`, `accent_color` and `loved_color` are the only settable palette colors; dividers, borders and placeholders are mixes of text and background (`render/color.ts`) using ratios measured from the presets. Names are **roles, not elements** - `meta` covers the timestamps, the footer and the stats labels together. `resolveTheme` returns the preset object itself when nothing is overridden, and a test asserts that by identity, so existing cards cannot drift.
- **`logo_color` is not part of `Theme`.** The wordmark is a trademark, so it defaults to Last.fm's red in every palette and lives as an option rather than a theme field.
- **`artist` and `meta` are two controls on purpose.** Deriving one from the other is within 6/255 on the neutral themes but out by 21 and 41 on `nord` and `catppuccin`, which pair a hued artist line with a neutral grey timestamp - the derivation turns that grey blue or purple. They were once merged into one `muted_color`, and it had to be undone; the configurator links them by default instead, a convenience limited to that page. The URL always carries both colors in full, so the Worker has no notion of linking and a hand-edited URL cannot reach a state the form cannot show.
- **`loved` is separate from `accent` only because of `nord`.** Its accent is blue and a blue heart reads as something else; the other five presets set the two to the same color. Don't "simplify" it away.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeffreyCA/lastfm-recently-played-readme](https://github.com/JeffreyCA/lastfm-recently-played-readme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
