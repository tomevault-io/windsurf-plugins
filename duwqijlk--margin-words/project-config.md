---
trigger: always_on
description: A static reader for English novels, for Chinese junior-high learners. It is a plain Vite + React 19 +
---

# Margin Words: project notes

A static reader for English novels, for Chinese junior-high learners. It is a plain Vite + React 19 +
Tailwind 4 + zustand single-page app. It has no AI calls. The reader works fully with no account.
Optional accounts (email, password, and sync of shelf, progress, saved words, and settings) are
Cloudflare Pages Functions in `functions/` plus a D1 database. See docs/ACCOUNTS.md. Do not store user
data in R2. Books and word lists come from **book packs** (see README.md, "Reader and book packs"). A pack is one `.zip` with exactly `book.epub` + `glossary.json` (docs/book-pack-spec.md, section 3; `src/lib/pack-check.ts`, used by the pack tools in `scripts/`). **The app has no file import**: books are added ONLY from the Discover page, and only by a signed-in reader (`canAddBooks` in `src/lib/can-add.ts`; signed-out visitors browse everything, the add button says "Sign in to add", and books already stored on the device stay readable). A word-list book takes the reader's own EPUB from its Discover card (the own-EPUB dialog with the 80% match check). A standalone EPUB is never imported as a new book.

## Where to work

- Develop, push, and open pull requests only in the public repository
  `https://github.com/duwqijlk/margin-words`.
- The private archive `https://github.com/duwqijlk/margin-words-archive` was archived by the owner on
  2026-10-04. It is read-only. Pull requests #1 through #34 stay there. #1 through #32 still contain
  copyrighted EPUB files, so that archive must stay private. Do not unarchive it, and do not make it public.
  Do not develop, push, or open pull requests there.
- The site is not connected to GitHub. The Cloudflare Pages project is `margin-words`. The domains are
  `https://inputread.site` and `https://www.inputread.site`. D1 and R2 do not follow the repository.
  Archiving the old repository does not take the site down.
- Copyrighted EPUBs live only in the private bucket `margin-words-private`. Public-domain EPUBs live on
  `https://books.inputread.site`. Do not put a third-party EPUB in git. The only EPUB in this history is
  `examples/sample-book/the-lantern-seller.epub`.
- Deploy stays manual, from this public repository: `npm run build` (leave `VITE_BOOKS_BASE` unset; an empty
  value is `npm run build:local`, for tests only), then
  `npx wrangler pages deploy dist --project-name margin-words --branch main`. That deploy uploads the built
  site (`dist/` and `functions/`). Do not upload `dist-books/`, `packs/`, or `dist-private/` with it.

## Rules

- The UI has two languages, `en` and `zh`. Every visible string (labels, aria-labels, toasts, errors) goes
  through `useT()` / `tr()` from `src/lib/i18n.ts` and lives in BOTH `src/lib/i18n-en.ts` (plain, simple English)
  and `src/lib/i18n-zh.ts` (simple Simplified Chinese for junior-high students). Both must have the same keys (tsc
  and `npm test` check this). Book content (meanings, paragraph/sentence help, phrases, titles) is English and is
  never translated.
- Chinese text is allowed ONLY in `src/lib/i18n-zh.ts`, `docs/` and `README.zh-CN.md`. No Chinese in other
  files of `src/`, `public/`, `packs/`, `examples/` or `scripts/`. `npm run check:cjk` must pass. (The Chinese
  text of the guide page in the app is in `docs/guide-chrome.json` and built into `dist/kit/`, never stored in `public/`.)
- Never change the reading layout when a panel opens. Panels are overlays. Test with a layout-shift run.
- The pack word lists (`packs/<id>/glossary.json`) are book content. Do not edit them by hand; copy them
  byte for byte. `node scripts/build-packs.mjs --check` must pass.
- No network call may depend on a server of ours except the optional account API on the same Pages
 host (`/api/*`, Pages Functions + D1). The reader may only fetch the catalog and pack files the user
 chose, plus that same-origin account API when someone is signed in. Book files stay on the books host.

## Commands

- `npm run dev` : dev server (also serves `/packs/*`, `/public-books/*` and `/word-lists/*` from the repo; book URLs stay on this origin)
- `npx vite build` (or `npm run build`) : static front end in `dist/` only (HTML, JS, CSS, fonts, guide). Book files are not in `dist/`. The build bakes `https://books.inputread.site` unless `VITE_BOOKS_BASE` is set. `npm run build:local` sets it empty so a preview server can serve the repo copies (used by e2e).
- `npm run build:books` : write `dist-books/`, the object keys to upload to the books bucket (loose `public-books/` epub, glossary, cover, catalog; loose `word-lists/` glossaries, card-sized `cover.jpg` when `packs/<id>/cover.jpg` exists, and catalog). No zip, no `all-packs.zip`, no copyrighted EPUB. A word-list book with no cover keeps the generated cover in the app.
- `npm run build:private` : write `dist-private/`, every copyrighted pack's EPUB (when it has one) plus its glossary and cover. Upload that folder to the private R2 bucket `margin-words-private`. That bucket has no public access. The app never fetches it. Never put `dist-private/` inside `dist/` or `dist-books/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duwqijlk/margin-words](https://github.com/duwqijlk/margin-words) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
