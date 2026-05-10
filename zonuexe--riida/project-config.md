---
trigger: always_on
description: This file records implementation details and operational context that are useful during development but do not belong in the user-facing README.
---

# AGENTS.md

## Purpose

This file records implementation details and operational context that are useful during development but do not belong in the user-facing README.

For a higher-level architecture and responsibility map, see [docs/design-doc.md](docs/design-doc.md).

## Release And Licensing

Release builds must satisfy the project's software licensing requirements.

In practice this means:

- the app's own license information must remain visible in the About dialog
- bundled third-party notices must be available from `THIRD-PARTY-LICENSES-rust.md` and `THIRD-PARTY-LICENSES-js.md`
- dependency license checks must pass before release or public distribution

When dependency sets change, update notices and run the checks before committing release-facing changes:

```bash
nix --extra-experimental-features 'nix-command flakes' develop --command npm run generate:third-party-licenses
nix --extra-experimental-features 'nix-command flakes' develop --command npm run check:licenses:npm
```

Combined local shortcut:

```bash
nix --extra-experimental-features 'nix-command flakes' develop --command npm run check:licenses
```

CI also checks the release gate:

- Rust dependency licenses via `cargo-deny` using [deny.toml](deny.toml)
- npm production dependency licenses via `license-checker`
- PR dependency review via [.github/dependency-review-config.yml](.github/dependency-review-config.yml)
- notice regeneration via [.github/workflows/license-check.yml](.github/workflows/license-check.yml)

If a dependency is added or updated, assume `THIRD-PARTY-LICENSES-rust.md` and `THIRD-PARTY-LICENSES-js.md` may need regeneration.

For macOS builds, the current default is ad-hoc signing via
[src-tauri/tauri.conf.json](src-tauri/tauri.conf.json).
This is intended for local verification and CI smoke testing only.
Public distribution still requires proper Apple signing and notarization.

## Design System

UI implementation references [DESIGN.md](DESIGN.md) for visual language, color roles, typography, and component patterns.

## Architecture

`riida` is a Tauri v2 desktop app with:

- Rust backend in [src-tauri/src/lib.rs](src-tauri/src/lib.rs)
- Vite/TypeScript frontend in [src/main.ts](src/main.ts)
- Styling in [src/styles.css](src/styles.css)
- Nix flake dev environment in [flake.nix](flake.nix)

The current app focus is local PDF library management with embedded reading, notes, thumbnails, and viewer preferences.

The library now also supports external, non-file-backed books such as Kindle purchases.

## Config, Data, Cache

The app separates storage into:

- config
- data
- cache

Current rules:

- Config prefers `~/.config/riida/riida.toml` when `~/.config` exists.
- Otherwise config falls back to the OS-native config directory.
- Data uses the OS-native app data directory.
- Cache uses the OS-native cache directory.

Legacy project-root files are migrated forward automatically:

- `riida.toml`
- `data/app.db`
- `data/thumbnails/`

When changing this logic, preserve migration behavior unless there is an explicit migration plan.

## Current Config Fields

Example development config in [riida.toml.example](riida.toml.example):

```toml
library_roots = ["~/Documents/Ebooks/"]
excluded_patterns = ["**/backup/**", "*.bak.pdf"]
pdf_renderer = "pdfjs"
```

Important fields:

- `library_roots`: recursively scanned and watched
- `excluded_patterns`: glob patterns matched against file names and paths
- `pdf_renderer`: `"pdfjs"` or `"native"`

## PDF Viewer Notes

There are two rendering paths:

- `native`: iframe/native WebView PDF display
- `pdfjs`: custom PDF.js renderer

The `pdfjs` path currently includes:

- text selection
- link overlays
- per-file and global viewer preferences
- target-page-prioritized rendering
- reading-position restore using `pageNumber + pageOffsetRatio`
- local cache in `localStorage` plus SQLite persistence

If you touch rendering order or page DOM structure, manually re-check:

- restore after changing viewer settings
- restore after reopening a file
- restore after back/forward navigation

## EPUB Viewer Notes

EPUB support is shipped as an **in-development feature**. On first open
the viewer shows a one-time notice warning that links may not work and
that the layout may break. The notice is gated by the
`riida.epub.previewNoticeShown` key in `localStorage`.

Rendering is done with [epub.js](https://github.com/futurepress/epub.js)
in paginated flow. Reading position is persisted via CFI
(`ReadingPosition.cfi`). Keyboard navigation is wired through a
top-level `window` `keydown` listener that checks `activeEpubRendition`
and calls `rendition.next()` / `rendition.prev()`. Keyboard focus
inside the iframe is handled by a `window.blur` → `setTimeout(0)` →
`window.focus()` refocus trick so that key events always reach the
top-level listener.

The remaining known issue is **link handling** — see below.

### Known Link-Handling Issue

Link clicks inside the EPUB iframe do not behave correctly in Tauri v2
on macOS (WKWebView). As of v0.2.3 the observable symptoms are:

- `https://` and `mailto:` links: clicking them produces **no visible
  reaction**. The system browser does not open; the viewer does not
  change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zonuexe/riida](https://github.com/zonuexe/riida) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
