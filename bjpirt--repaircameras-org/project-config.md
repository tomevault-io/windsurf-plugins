---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

repaircameras.org is a static site built with Eleventy (11ty) that hosts camera repair manuals and resources. The site uses TypeScript, JSX/TSX for templates, and SCSS for styling.

## Common Commands

```bash
# Start site development server with live reload
npm start

# Build the site for production
npx @11ty/eleventy

# Admin UI development
npm run admin:dev       # Start admin dev server + OAuth proxy
npm run admin:build     # Type-check and build admin app
npm run admin:test      # Run admin unit tests (vitest)

# Admin commands (from admin/ directory)
cd admin
npx vitest run          # Run unit tests once
npx vitest              # Run unit tests in watch mode
npx tsc --noEmit        # TypeScript check only
npx playwright test     # Run e2e tests
```

## Architecture

### Content Structure

- **Site content**: `site/` directory
  - Camera pages: `site/cameras/{manufacturer}/{model}.md` - Markdown files with frontmatter
  - Manufacturer indexes: `site/cameras/{manufacturer}/index.md`
  - PDF manuals: `site/files/{manufacturer}/{filename}.pdf` - Service manuals, repair guides, parts catalogs
    - Organised in manufacturer subfolders (e.g., `site/files/pentax/pentax-mx-service-manual.pdf`)
    - Naming convention: `{manufacturer}-{model}-{document-type}.pdf`
    - Use kebab-case for all components
  - Static assets: `site/static/`

### Camera Page Frontmatter

Camera markdown files use this structure:
```yaml
---
layout: item.11ty.tsx
tags:
  - cameras
manufacturer: Pentax
model: MX
relatedFiles:
  - pentax/pentax-mx-service-manual  # {manufacturer}/{filename} without .pdf
relatedLinks:
  - pentax-k1000-youtube      # ID from site/_data/links/
troubleshooting:
  - symptom: Shutter stuck
    cause: Old lubricant
    solution: CLA needed
---
```

### Templates and Components

- **Layouts**: `_layouts/` - TSX templates using `.11ty.tsx` extension
  - `item.11ty.tsx` - Individual camera pages
  - `manufacturerIndex.11ty.tsx` - Manufacturer listing pages
  - `mainIndex.11ty.tsx` - Main cameras index
  - `content.11ty.tsx` - Generic content pages

- **Components**: `components/` - Reusable TSX components
  - `MainTemplate.tsx` - Base HTML template with header/footer/breadcrumbs
  - `ResourceLink.tsx` - Displays PDF/link thumbnails
  - `Breadcrumbs.tsx` - Navigation breadcrumbs

- **Component imports**: Use `@components/*` path alias (configured in tsconfig.json)

### Data Pipeline

The site uses Eleventy's global data system (`site/_data/`) to process PDFs and external links:

- **`files.js`**: Recursively scans `site/files/{manufacturer}/` subdirectories, extracts metadata (title, description) from PDF properties, generates thumbnails at build time, stores in `_site/img/thumbnails/`
- **`links.js`**: Reads JSON files from `site/_data/links/`, pairs with corresponding JPG thumbnails, processes images for display

These data files are available globally in all templates as `files` and `links` objects. Files are keyed by `{manufacturer}/{filename}` (e.g., `pentax/pentax-mx-service-manual`). Links are keyed by filename without extension.

### Build Configuration

- **`eleventy.config.js`**: Custom extensions for TSX/SCSS compilation using tsx and sass packages
- **`tsconfig.json`**: JSX configuration with `jsx-async-runtime` for async component rendering
- Output directory: `_site/` (default Eleventy output)

### Shared Library (`lib/`)

- Type definitions: `lib/types/` — `File.ts`, `Link.ts`, `PageMetadata.ts`, `ImageMetadata.ts`, `cameraPage.ts`
- `lib/frontmatter.ts` — YAML frontmatter parse/stringify for camera markdown files
- Shared between site (via tsx at runtime) and admin (via `@shared/*` path alias)

### TypeScript

- No compilation step needed for site — tsx handles TypeScript at runtime
- Admin uses Vite with `tsc -b` type-checking before build

### Admin UI (`admin/`)

A React single-page app for managing site content via GitHub's API. Built with Vite, deployed to `/admin/`.

#### Architecture

- **React 19** with **React Router 7** for client-side routing
- **GitHub OAuth** authentication via Cloudflare Worker (`cloudflare-worker/`)
- Fork/PR workflow: changes are saved to a fork branch, then submitted as a pull request

#### Key Patterns

- **State management**: `useReducer` with dedicated reducer files (`editorReducer.ts`, `cameraEditorReducer.ts`). Each reducer has a companion `buildAndValidate()` function using Zod schemas
- **Data loading hooks**: `useTutorialLoader.ts`, `useCameraLoader.ts` — handle auto-resume from existing fork branches
- **GitHub service layer**: `github.ts` (shared Git tree API utilities), `github-camera.ts` (camera-specific operations)
- **Path alias**: `@shared/*` maps to `../lib/*` (configured in both `tsconfig.json` and `vite.config.ts`)

#### Structure

```
admin/src/
├── components/       # Reusable UI (PdfPicker, TroubleshootingEditor, LinkCreator, AnnotationEditor, PhotoManager)
├── hooks/            # useAuth
├── pages/            # Route components (TutorialList/Editor, CameraList/Editor, StepEditor) + reducers + loaders

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bjpirt/repaircameras.org](https://github.com/bjpirt/repaircameras.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
