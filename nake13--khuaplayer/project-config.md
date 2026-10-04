---
trigger: always_on
description: Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.
---

# Prototype Instructions

Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

Before making substantial visual changes, use the Product Design plugin's `get-context` skill when the visual source is unclear or no longer matches the current goal. When the user gives durable prototype-specific design feedback, preferences, or decisions, record them in `AGENTS.md`.

When implementing from a selected generated mock, treat that image as the source of truth for layout, component anatomy, density, spacing, color, typography, visible content, and hierarchy.

Build app UI in `src/`. Keep `.openai/hosting.json`, `worker/index.js`, `scripts/prepare-sites-build.mjs`, and `tests/sites-worker.test.mjs` intact so the same local prototype can be handed to Sites. Before a Sites handoff, run `npm run build` and `npm run test:sites`; the build must leave `dist/client/index.html`, `dist/server/index.js`, and `dist/.openai/hosting.json`.

## Khua website direction

- The product name on the site is “Khua Player” (wordmark, page title, meta, footer, closing headline, download buttons, labels) so it is not misread; body copy may use “Khua” as the short form after that.

- Source visual: `reference/final-design.png` (composition and particle language only; its ultra-condensed type was superseded).
- Keep the site self-contained under `Website/`; never copy app binaries into this folder or Git history.
- Use the real Khua icon from `public/assets/khua-icon.png`.
- Visual language: bone-white editorial canvas, monumental sentence-case type, electric cobalt/cyan particle velocity fields, and tiny vermilion registration accents. Avoid flat saturated color slabs, boxed number badges, and all-caps mono everywhere; those read cheap.
- Typography relies on fonts that ship with macOS (the audience is Mac users; other platforms get plain fallbacks): display is Helvetica Neue Bold with tight tracking, sentence case; exactly one accent word per headline is Bodoni 72 Book Italic (marked `*word*` in `src/content.js`); body copy is the system font (SF Pro Text via `-apple-system`; Iowan Old Style is the approved serif alternative if an editorial voice is wanted); navigation, buttons, eyebrows, and small labels use the system font (SF Pro via `-apple-system`), eyebrows as tracked uppercase; monospace (SF Mono via `ui-monospace`) appears only on diagram tech tags. Chinese uses PingFang SC for display, body, and UI, with the accent word in Songti SC Black. No web-font packages are loaded.
- Buttons and controls are pill-shaped and sentence case. The header is fixed with a frosted-glass background and flips to a dark glass automatically over dark or blue sections.
- The performance diagram is a single Apple-style isometric stack showing the path a frame takes: the file as a flat card, Khua Player, Metal · VideoToolbox, a glowing zero-copy band, Apple silicon. No comparison with “other players” (unverifiable, and a competitor comparison by implication). Labels stay one or two words; the caption carries the explanation and keeps “Wherever the format allows”, because unsupported formats use the FFmpeg/dav1d software path.
- Match all 17 languages in the app's `Localizable.xcstrings` catalog. Use a native language picker on both the homepage and release history showing only the 17 actual languages, with the currently displayed language selected. The user explicitly rejected an Automatic option for the website: language detection is background behavior, not an exposed setting. Resolve language from an explicit `?lang=`, then a saved manual choice, then supported browser languages, then a conservative Cloudflare country hint, then English. Never let IP override any matching browser language or explicit choice. Accept legacy `zh` links as `zh-Hans` and distinguish `zh-Hant`. Only manual picker actions may save preferences; shared links are visit-only, and inferred languages must not be saved or inserted into the URL. Automatically detected visits keep clean URLs; manual/shared-language links retain their language. Ignore the old mixed-purpose `khua-site-locale` key. Copy lives in `src/locales/<locale>.json`; keep all versions natural rather than mechanically literal. Run `npm run test:i18n` to verify complete copy, placeholders, app-language parity, release-note coverage, preference changes and async races.
- Only query `/api/locale` when no browser language matches. Read the trusted `request.cf.country` at the edge and return a supported locale or null, not IP/location details. No third-party geolocation, cookies, database writes, visitor logs or shared response caching. Limit the optional lookup to one second; offline, timeout and uncertain/multilingual regions keep English. Cancel late results after manual selection, navigation or browser-language changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nake13/khuaplayer](https://github.com/nake13/khuaplayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
