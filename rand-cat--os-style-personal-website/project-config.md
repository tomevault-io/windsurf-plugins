---
trigger: always_on
description: This repo is an Astro-based personal website. The home page is not a standard landing page; it behaves like a small desktop OS with draggable windows, desktop icons, a blog app, and supporting app-like panels.
---

# AGENTS.md

## What This Project Is

This repo is an Astro-based personal website. The home page is not a standard landing page; it behaves like a small desktop OS with draggable windows, desktop icons, a blog app, and supporting app-like panels.

The project has two equally important parts:

- a desktop-style shell on `/`
- a conventional content-backed blog system under `/blog`

When making changes, treat the desktop interactions as product behavior, not decorative chrome.

## Stack

- Framework: Astro
- Content: Astro Content Collections
- Main content source: `src/content/blog`
- Client behavior: browser JavaScript loaded from `public/scripts`
- Styling: CSS loaded from `public/styles`

Useful commands:

- `npm run dev`
- `npm run build`
- `npm run preview`
- `npm run import:jike -- "/absolute/path/to/export.csv"`
- `npm run jike:translate:worklist -- --limit 20`

## First Places To Read

If you are new to the repo, start here:

1. `src/pages/index.astro`
   The main desktop-style home page. This is the integration point for the OS UI.
2. `src/data/desktop.ts`
   Desktop app metadata, icon labels, group previews, window sizes, desktop icon positions, and external links.
3. `src/components/desktop/WindowFrame.astro`
   Shared shell for both standard desktop windows and special group-style overlays.
4. `src/components/desktop/DesktopBlogWindow.astro`
   The main blog application shown inside the desktop.
5. `src/lib/blog.ts`
   Shared blog helpers: locale logic, paths, formatting, and post lookup.

Useful desktop-specific components to inspect immediately:

- `src/components/desktop/DesktopIcons.astro`
- `src/components/desktop/DesktopDock.astro`
- `src/components/desktop/VibaryWindow.astro`
- `src/components/desktop/ProductPlaceholderWindow.astro`
- `src/components/desktop/SocialFolderWindow.astro`
- `src/components/desktop/JikeIdeasWindow.astro`

## Blog Architecture

The blog exists in two forms:

- standard pages under `src/pages/blog/**`
- an embedded reader used inside the desktop window

Important files:

- `src/components/blog/BlogIndexPage.astro`
- `src/components/blog/BlogArticlePage.astro`
- `src/components/blog/BlogEmbedPage.astro`
- `src/content/blog/*`
- `public/scripts/locale-preference.js`

If blog typography, locale switching, or article selection feels wrong, inspect both the desktop blog window and the underlying blog page components.

Current locale behavior:

- Blog routes remain path-based:
  - `/blog` for Chinese
  - `/blog/en` for English
- On first visit, if `localStorage["blog-locale"]` is missing, the client now detects `navigator.languages` and stores either `zh` or `en`.
- After the user manually switches once, the stored locale takes priority over browser language.
- The same stored locale is reused across:
  - desktop blog
  - content-app windows
  - Jike window localized UI
  - desktop cat speech bubble copy

## Desktop Architecture

The home page is assembled from desktop-specific components plus shared window behavior.

Core ideas:

- each app has an id defined in `src/data/desktop.ts`
- the page uses that metadata to render icons and windows
- shared window behavior should stay in the window shell / desktop script layer instead of being duplicated per app

Desktop-related files are mainly under:

- `src/components/desktop/`
- `src/data/desktop.ts`
- `public/styles/desktop-os.css`
- `public/scripts/desktop-os.js`

Desktop CSS and JS are intentionally split into modules now. Keep them split by concern instead of pushing everything back into one file.
If a window/app has custom UI that is not clearly reusable, give it its own CSS file instead of growing `panes.css`.

CSS entry:

- `public/styles/desktop-os.css`

CSS modules:

- `public/styles/desktop/shell.css`
  Page shell, desktop icons, top bar, wallpaper/background treatment, and global desktop states.
- `public/styles/desktop/windows.css`
  Standard window shell, focus states, controls, maximize/fullscreen behavior, and group overlay shells.
- `public/styles/desktop/panes.css`
  Shared pane styles only. Do not keep app-specific window styling here unless it is genuinely reused.
- `public/styles/desktop/dock.css`
  Dock tray, magnification styling, active dots, hover labels, and show/hide behavior.
- `public/styles/desktop/responsive.css`
  Responsive adjustments and small-screen fallbacks.
- `public/styles/desktop/*.css`
  App-specific window styling such as `jike.css`, `studio.css`, `settings.css`, `read-easy.css`, `vibary.css`, and related per-window files.

JS entry:

- `public/scripts/desktop-os.js`

JS modules:

- `public/scripts/locale-preference.js`
  Shared client-side locale preference helpers: first-visit browser-language detection, storage reads/writes, and locale normalization.
- `public/scripts/desktop/window-manager.js`
  Open/close/focus/minimize/maximize, initial window placement, group overlay open/close, drag, and 8-direction resize.
- `public/scripts/desktop/dock.js`
  Dock magnification and slot-width behavior.
- `public/scripts/desktop/blog-browser.js`
  Blog locale switching, embedded article routing, and sidebar behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rand-Cat/os-style-personal-website](https://github.com/Rand-Cat/os-style-personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
