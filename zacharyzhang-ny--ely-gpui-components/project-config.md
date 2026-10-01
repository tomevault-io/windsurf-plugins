---
trigger: always_on
description: Ely GPUI Component. A component library for GPUI, in light and dark.
---

# AGENTS.md

Ely GPUI Component. A component library for GPUI, in light and dark.

## Stack

- Rust 1.95, edition 2024. One crate: `ely-gpui-component`.
- gpui and gpui_platform from Zed's repo by git at 1a28cff, the code gpui-pre 0.3.7 republishes. gpui takes `stacker`, which keeps deep layouts off the stack's end; gpui_platform takes `font-kit`, without which macOS draws no text. Feature `runtime_shaders`, on by default, turns on gpui_platform's: machines without Xcode lack the Metal compiler.
- Assets, embedded with `rust-embed`: Lucide 1.48.0 icons (ISC), Inter 4.1 (regular, medium, semibold, italic), JetBrains Mono 2.304 and IBM Plex Sans Regular (OFL). Plex sits unmodified, its name reserved, where gpui's svg renderer loads it: gpui_web answers `.SystemUIFont` with it, and svg text falls back to it on the web.
- Catalog data: `emojis` (Unicode emoji), `isolang` (ISO 639, native names), `isocountry` (ISO 3166), `iso_currency` (ISO 4217, also minor units for money).
- Codes: `qrcode` (QR) and `barcoders` (Code 128, EAN-13), both MIT OR Apache-2.0, default features off. `rqrr` 0.11 ((MIT OR Apache-2.0) AND ISC, default features off) reads QR codes from the host's frames.
- macOS extras (tray icon, Dock badge) call AppKit through `cocoa` 0.26 and `objc` 0.2, the crates gpui already links.
- Web views: `wry` 0.57 (Apache-2.0 OR MIT, default features off, macOS only) lays a child WKWebView over the window.
- Terminal: `alacritty_terminal` 0.26 (Apache-2.0, default features off, native only) for the grid, its parser and the pseudo-terminal; `futures` carries its events. `vte` 0.15 (Apache-2.0 OR MIT), the parser it re-exports, reads `AnsiText`'s codes alone, so colored output builds for wasm32.
- Diffs: `similar` 3.2 (Apache-2.0) for line and word diffs and three-way merges; its `unicode` feature splits words at punctuation.
- Markdown: `pulldown-cmark` 0.13 (MIT, default features off) for CommonMark with tables, tasks, strikethrough, footnotes and math.
- Pictures: `image` 0.25 (MIT OR Apache-2.0), the crate gpui decodes with, default features off; it turns decoded frames and rims map tiles.
- Maps: Natural Earth's 110m countries (public domain). `assets/maps/countries.geojson`, trimmed by `scripts/countries.py` to an ISO code, a name and positions to 0.01°, backs `maps::WorldMap`; the gallery's tiles are cut from the same file, z0 to z3, by `scripts/tiles.py` with Pillow.
- Web: on wasm32, `web-time` 1.1 (MIT OR Apache-2.0) gives the `Instant` gpui's executor returns, std's own natively; jiff takes `js`, rust-embed `debug-embed`, similar `wasm32_web_time`. The gallery's web entry takes `gpui_web`, `wasm-bindgen`, `js-sys` and `web-sys` as wasm-only dev-dependencies, and a Hebrew cut of Noto Sans Hebrew 2.003 (OFL, `examples/gallery/fonts`): gpui_web asks the browser for emoji and CJK alone. The `wasm-bindgen` CLI at Cargo.lock's version and binaryen's `wasm-opt` finish the bundle.
- Gallery: `examples/gallery`. Website: `frontend/` (Vite 8.3, pnpm 11, TypeScript, no framework): a home page in motion with three.js 0.186 and Lenis 1.3, and a components page with chapters on the left and the chosen story on the right, after astryx.atmeta.com and gpui-kit.com. Each story runs live from the gallery built for wasm32 through gpui_web, in a same-origin iframe, never as a screenshot. Geist and Geist Mono (OFL, `@fontsource-variable`), Reicon icons (MIT, based on Solar Icons, CC BY 4.0, credited in the footer), simple-icons' GitHub mark (CC0). It deploys to Cloudflare Workers static assets through `npx wrangler` (4.143).

## Commands

- `cargo run --example gallery` opens the gallery. `-- --page <slug>` starts on a page; `--story <title or slug>` with it draws that section alone, and `--capture` then shoots it without the page's script.
- `cargo run --example gallery -- --capture <dir>` writes PNGs of every page, light and dark, top to bottom, then each page's scripted states, including windows the demos open. macOS only.
- `cargo run --example gallery -- --narrow 280 --capture <dir>` lays each page, header and body, at that width in a padded card and shoots it without its script, whose aims are measured at full width. Demos that set their own width run past the card.
- `RUSTC_BOOTSTRAP=1 cargo check --lib --target wasm32-unknown-unknown` checks the web build; gpui_web's `wasm_thread` needs the bootstrap, as Zed's web examples do.
- `ELY_GALLERY_ASSETS=<address of out/assets/> scripts/web.sh <out>` builds the gallery for the browser, profile `web`, and fails past Cloudflare's 25 MiB. It opens at `?page=<slug>&story=<title or slug>&theme=light|dark`; the host page calls `gallery.setTheme(mode)` or posts `{ ely: "theme", theme }`, before the wasm runs or after, and the page posts `{ ely: "ready" }` to its parent once it runs.
- In `frontend/`: `ELY_GALLERY_ASSETS=<site>/gallery/assets/ pnpm gallery` builds the gallery into `public/gallery`, `pnpm dev` serves the site, `pnpm build` checks types and writes `dist`, `pnpm deploy` runs `wrangler deploy`. `node scripts/og.mjs`, with `pnpm preview` up, renders `public/og.png` from the hero. The site reads `examples/gallery/stories.json` for its chapters, stories and search.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZacharyZhang-NY/Ely-GPUI-Components](https://github.com/ZacharyZhang-NY/Ely-GPUI-Components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
