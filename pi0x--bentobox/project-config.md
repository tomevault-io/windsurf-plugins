---
trigger: always_on
description: Bentobox — an Apple-style bento grid generator. Vite + React 19 + TS + Tailwind v4.
---

# AGENTS.md

Bentobox — an Apple-style bento grid generator. Vite + React 19 + TS + Tailwind v4.
`pnpm dev` (the user usually has it running), `pnpm build`, `npx tsc --noEmit -p tsconfig.json`.

## Layout

```
src/bento/BentoCanvas.tsx  the board — one tree, used for BOTH preview and export
src/bento/render.tsx       Takumi export: PNG / WebP / SVG + clipboard copy
src/bento/markdown.ts      the board as a Markdown document, both ways (md4x)
src/bento/share.ts         the board as a URL fragment, both ways (base64url JSON)
src/bento/icons.ts         emoji vs. Iconify: fetch, cache, ink, search
src/bento/liquid-glass.ts  the refraction itself: raw RGBA in, raw RGBA out
src/bento/glass.tsx        that pass wired to a board — backdrop, rects, layer
src/bento/layout.ts        which cell each card lands in, and its rect in pixels
src/bento/{types,themes,presets}.ts   model, palettes, templates + canvas sizes
src/ui/{Preview,Sidebar,Field,AppearanceToggle}.tsx  app chrome; App.tsx owns all state
```

## Rules that keep this working

- **`BentoCanvas` is inline styles only.** It is mounted in the DOM *and* handed to
  Takumi's Rust layout engine; a stylesheet would be one more thing to keep in sync.
  Anything preview-only (selection ring, click target) must be gated on the
  `onSelect` prop so it cannot leak into an export.
- **App chrome is Tailwind, the board is not.** Don't mix.
- **Dark mode is chrome-only.** `app.css` redefines the `dark:` variant to follow
  `data-theme` on `<html>`; `useAppearance` mirrors the light/dark choice onto
  that attribute and the inline script in `index.html` resolves the same value
  before first paint, so a reload doesn't flash white — keep the two in sync. The
  OS preference only seeds the first run; after that the toggle is the only input.
  The board keeps its own `config.themeId`, so an export looks the same on any
  machine — never wire the appearance setting into `BentoCanvas`.
- **Takumi runs as WASM in the browser** — Vite's `browser` condition resolves
  `takumi-js` to it and pulls the 3.5 MB binary via a `?url` import. That import is
  why `vite.config.ts` has `optimizeDeps.exclude`; keep it.
- **`format` is a discriminated union** in Takumi's render options — pass a literal
  per branch, not a variable.
- **Takumi's `width`/`height` are output pixels, not CSS pixels.** `devicePixelRatio`
  scales the board's CSS pixels into that canvas, so a 2x export asks for a 2x
  canvas too (`render.tsx`). Passing the board's own size alongside `dpr: 2`
  silently renders the top-left quadrant at double scale and crops the rest.
- **An icon is an emoji or an Iconify name.** Emoji go through Takumi's emoji
  provider as glyphs — Geist (built into Takumi) has no dingbats, and a text
  presentation selector (`U+FE0E`) rasterizes as tofu, so add to `ICON_CHOICES`
  in `types.ts` only after rendering the glyph and looking at it. Anything
  shaped `prefix:name` is an Iconify SVG instead (`icons.ts`): fetched once into
  a module-level cache, `currentColor` swapped for the card's own ink, handed to
  both engines as an `<img>` data URI. `BentoCanvas` has to stay synchronous, so
  a cache miss falls back to the API's URL — `exportBento` awaits `preloadIcons`
  first so an export embeds bytes and not a link.
- **The Markdown tab round-trips through `markdown.ts`, and that has to stay
  exact.** `toMarkdown(fromMarkdown(doc))` is what lets the tab re-serialize on
  every keystroke without stealing the caret (`describes()` in `Sidebar.tsx`).
  Anything a card can hold must therefore be spellable: the escape hatches
  (`icon=""` for an emoji-led title, `icon="mdi:rocket"` for an icon with no
  glyph to lead with, `stat=` / `unit=`, an `&nbsp;` heading for a card with no
  headline, `### ` for a stat card's title) exist only for that.
  Add a field to `BentoCard` and you owe it a slot here — the smoke test below
  catches what you forget.
- **The URL fragment is the board's only persistence, so a link has to keep
  meaning what it meant.** `share.ts` writes the *whole* `BentoConfig` — never a
  diff against `defaultConfig`, which would halve the URL and silently repaint
  every link ever shared the day a template changes. Card ids are dropped (React
  keys, not content) and minted fresh on the way in; the payload carries a `v`
  and a decoder that doesn't recognise it returns `null`, meaning "leave the
  board alone", never "reset it". It is JSON rather than the Markdown of
  `markdown.ts` because `fromMarkdown` needs md4x's WASM: a board that can only
  arrive after an await flashes the default template first, and this one decodes
  in the `useState` initialiser.
- **Nothing arriving from the URL is trusted, and nothing about it touches the
  History API.** Every decoded value is clamped to `LIMITS`, matched against a
  closed set from `types.ts`, or dropped — those live next to the model precisely
  so the sidebar's fields and both parsers can't drift apart. `App.tsx` writes the
  hash with a debounced `location.replace`: a fragment-only replace stays on the
  page and leaves the history stack alone, where assigning `location.hash` would
  push an entry per keystroke. It does fire `hashchange`, so the listener that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pi0x/bentobox](https://github.com/pi0x/bentobox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
