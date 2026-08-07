---
trigger: always_on
description: `astroprint` is an Astro integration for Markdown-first documents with normal web preview, Paged.js browser preview, and PDF export.
---

# AGENTS.md

## Project Overview

`astroprint` is an Astro integration for Markdown-first documents with normal web preview, Paged.js browser preview, and PDF export.

Calling `astroprint()` with no options should only install the Markdown processing pipeline: directives, the built-in `:logolink` transform, BibTeX conversion, and HTML comment stripping. Do not inject routes unless the user explicitly configures `injectedRoutes`, and do not set a default PDF target unless the user configures top-level `pdf`. `injectedRoutes` is a list, not a keyed object; PDF generation is configured separately through top-level `pdf`.

The integration should update Vite config so `vite.server.watch.ignored` includes `**/.astroprint*/**`. Preserve caller-owned watcher ignores via Astro/Vite config merging, and do not re-add the exact pattern when it is already configured.

Route injection should be decided before calling `injectRoute`, not inside generated route `getStaticPaths()`. Generated routes should assume they are meant to render once injected. In `astro dev`, always inject configured normal routes. In PDF render builds, `ASTROPRINT_RENDER_HTML=true` must force normal route injection regardless of route flags so `astroprint pdf` can reach configured routes. In normal `astro build`, each route's `injectDuringBuild` flag controls normal route injection; it defaults to `true`. Preview routes are opt-in only: omit `previewRoute` or set `previewRoute: false` to skip preview route injection, set `previewRoute: true` to use the default `${route}-preview` path (`/preview` for `/`), or pass a custom preview route string.

Injected route configs must include an explicit `route`; astroprint should not guess a public URL. Supported sources are `markdown` for a single Markdown file, `collection` plus `entry` for one content collection item, and `collection` plus optional `defaultId` for a multi-document collection route. Keep these source shapes mutually exclusive in TypeScript. PDF output paths are resolved as normal filesystem paths: `outputDir` is the base directory and `output` is resolved inside it, with absolute `output` paths used as-is. When the CLI omits `--port`, the temporary server should bind to an OS-assigned free port.

`astroprint pdf` builds temporary HTML into `.astroprint/`. After the Astro build, write `.astroprint/.gitignore` with `*` because the build may recreate the output directory; do not unignore that file from inside itself. Validation directories such as `.astroprint-check/` remain maintainer-managed.

The package code lives in `src/`. Built-in Astro surfaces live directly under top-level source folders:

- `src/components/Document.astro` is the theme-neutral default document root. It should not render title markup or assume frontmatter fields.
- `src/components/AcademicDocument.astro` is the built-in academic document surface. It imports the academic CV theme, renders academic title markup, and wraps slotted content in `Document.astro`.
- `src/components/PreviewShell.astro` is the theme-neutral preview shell with navigation, print button, preview status, scroll restoration, and normal/Paged.js preview branching.
- `src/layouts/BaseLayout.astro` is the minimal HTML shell with `<html>`, `<head>`, viewport metadata, optional `pageTitle`, and global page/body baseline.
- `src/layouts/AcademicLayout.astro` is the built-in academic layout. It maps frontmatter/entry fields and switches between plain `AcademicDocument.astro` and `PreviewShell.astro + AcademicDocument.astro` with its `withPreviewShell` prop.
- `src/components/PrintPreview.astro` is the document-agnostic Paged.js preview wrapper.
- `src/styles/base.css` defines the required baseline page, typography, and preview CSS variables plus neutral document root styles.
- `src/styles/academic-cv.css` is the built-in academic CV document theme.
- `src/vendor/pagedjs-0.4.3.esm.min.js` is the vendored minified Paged.js ESM bundle used by `PrintPreview.astro`.

The playground content lives under `playground/` and is useful for local validation.

## Markdown Directives

`remark-directive` is installed by the integration. `remarkAstroPrintDirectives` should keep directives generic: known list aliases map to semantic tags (`ul`, `ol`, `li`, `entry`), unknown text directives default to `span`, and unknown leaf/container directives default to `div`. All directives get a default `astroprint-{name}` class unless the caller overrides that directive with `directives`.

Directive attributes should pass through to rendered HTML properties. Prefer standard directive attribute syntax for classes:

```md
:::::ul{.two-col}

::::entry
...
::::

:::::
```

Do not require a temporary `directives` entry in `astro.config.mjs` for the built-in two-column list styling. `[two-col]` is directive label/content syntax, not the preferred way to express a class.

The `:logolink[...]` directive is handled by `src/lib/remark-logo-link-directives.ts` before the generic directive mapper. Keep specialized transforms like this separate from the generic mapper when they need to rewrite the Markdown AST.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomiechen/astroprint](https://github.com/atomiechen/astroprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
