---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Approach
- Think before acting. Read existing files before writing code.
- Be concise in output but thorough in reasoning.
- Prefer editing over rewriting whole files.
- Do not re-read files you have already read unless the file may have changed.
- Test your code before declaring done.
- No sycophantic openers or closing fluff.
- Keep solutions simple and direct.
- User instructions always override this file.

## Output
- Return code first. Explanation after, only if non-obvious.
- No inline prose. Use comments sparingly - only where logic is unclear.
- No boilerplate unless explicitly requested.

## Code Rules
- Simplest working solution. No over-engineering.
- No abstractions for single-use operations.
- No speculative features or "you might also want..."
- Read the file before modifying it. Never edit blind.
- No docstrings or type annotations on code not being changed.
- No error handling for scenarios that cannot happen.
- Three similar lines is better than a premature abstraction.

## Review Rules
- State the bug. Show the fix. Stop.
- No suggestions beyond the scope of the review.
- No compliments on the code before or after the review.

## Debugging Rules
- Never speculate about a bug without reading the relevant code first.
- State what you found, where, and the fix. One pass.
- If cause is unclear: say so. Do not guess.

## Pipeline Rules
- NEVER pause for individual mockup placement preview. Always use saved calibration positions from mockup-positions.json and compose mockups automatically.
- Mockup positions are selected in bulk via the calibration screen, not one-by-one during pipeline.

## Simple Formatting
- No em dashes, smart quotes, or decorative Unicode symbols.
- Plain hyphens and straight quotes only.
- Natural language characters (accented letters, CJK, etc.) are fine when the content requires them.
- Code output must be copy-paste safe.
- User-facing UI/error strings are in Turkish without diacritics (e.g. "Cookie verisi gerekli"). Match that style when adding new ones.

## Commands

Setup: copy `.env.example` to `.env` and `config.example.json` to `config.json`, then enter `OPENROUTER_API_KEY` and Etsy keystring/shared secret/OAuth in the in-app Ayarlar screen. Browser/CDP settings are only needed for optional EtsyHunt/Pinterest work. Listing metadata is discovered from the shop or can be set with the optional Etsy defaults in `.env`.

- `npm run dev` - launch the CDP-controlled browser, then start the Express server on `:3000`
- `npm start` - server only; official Etsy API upload does not need a browser
- `npm run browser` - just launch Opera/Chromium with `--remote-debugging-port` per `config.json`
- `npm run create -- --ref <img> --mockups <m1,m2> --competitor <url> --sku <sku>` - one-shot CLI pipeline (see `create.js` for all flags: `--design`, `--prompt`, `--design-only`, `--skip-upload`, `--skip-tags`)
- `npm run audit-health` - run the 100-point listing health rubric across active listings; writes `reports/listing-health-{date}.{json,md}`
- `npm run pnl` / `npm run scrape-customhub` / `npm run scrape-printnest` / `npm run scrape-ehunt` - operational scrapers and P&L
- `npm run daily` / `npm run weekly` / `npm run monthly` - generate the operating reports under `etsy-projects/ETSY-Claude/` and `etsy-projects/ETSY-Aylin/`
- `npm run rules-excel` / `npm run holidays` / `npm run x-digest` - utilities under `etsy-rules/`

There is no test suite, linter, or build step. Verify changes by running the affected script or hitting the relevant endpoint.

## Architecture

Two entry points share the same `lib/` pipeline:

- **`server.js`** - Express 5 web app (the original wizard UI in `public/baby-puzzle.html` is served at `/` and `/legacy`; `/index.html` redirects to `/`). The creation API accepts only `single`, `front-back`, and `bulk` modes. Endpoints fall into three groups: cookie storage (file-based at `data/cookies.json`), pipeline operations (`/api/create`, `/api/regenerate-mockup`, `/api/test-tags`, `/api/generate-{tags,title,description}-ai`, `/api/mockup-to-video`), and listing helpers (`/api/sections`, `/api/popular-now`, `/api/suggest-personalization`). Static dirs `designs/`, `output/`, `mockups/` are auto-created and served. `lib/database.js` (sqlite + bcrypt) exists but is not wired into `server.js`; the live app is auth-less.
- **`create.js`** - same pipeline as a single-SKU CLI: generate -> compose mockups -> scrape tags -> upload -> pin.

Pipeline modules in `lib/`:
- `generate-design.js` - calls OpenRouter (`google/gemini-2.5-flash-image`) with the reference image; saves PNG to `designs/`.
- `compose-mockup.js` - Sharp-based composite. Position per-template stored in `mockup-positions.json` (keyed by template basename). Variants exist for flux/copyrighted/front-back; `detect-positions.js` auto-detects the shirt area when no position is set.
- `scrape-tags.js`, `scrape-tags-etsyhunt.js` - competitor/EtsyHunt scrapers + `optimize.js` (description, tag optimization, alt texts via OpenRouter).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitalvendorxx/etsy-product-creator-v2](https://github.com/digitalvendorxx/etsy-product-creator-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
