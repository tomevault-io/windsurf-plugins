---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Slappy - A Nuxt 4 application for generating printable name tag labels in the TownStix US-10 format (4×2" labels, 10 per sheet). This is a Vue 3/Nuxt migration of a Next.js application, using modern glassmorphism design patterns.

**Key Technologies:**

- **Nuxt 4** + Vue 3, **@nuxt/ui** (Tailwind), **nuxt-feature-flags**, **Pinia**, **Puppeteer**
- **pnpm** (version enforced via `packageManager` field)

See [README.md](README.md#technology-stack) for complete stack details.

## Development Commands

```bash
# Development
pnpm dev                  # Start dev server at http://localhost:3000
pnpm build               # Build for production
pnpm preview             # Preview production build
pnpm generate            # Generate static site

# Code Quality
pnpm lint                # Check with ESLint
pnpm lint:fix            # Auto-fix ESLint issues
pnpm format              # Format with Prettier
pnpm format:check        # Check formatting
pnpm deadcode            # Find unused code with Knip

# Combined
pnpm check:all           # Run lint + format:check + deadcode
pnpm fix:all             # Run lint:fix + format

# Dependencies
pnpm outdated            # Check for outdated deps
pnpm update:interactive  # Interactive update UI
```

## Architecture Overview

### Three-Step Wizard Flow

The application is built around a multi-step wizard:

1. **Upload Step** - CSV file upload or Google Sheets URL input
2. **Mapping Step** - Map CSV columns to name tag lines (line1/line2/line3)
3. **Preview Step** - Interactive HTML preview with zoom, and PDF export

### State Management Pattern

The app uses **composables** (not Pinia stores) for shared state:

- `useWizardNavigation()` - Controls step transitions (upload → mapping → preview)
- `useDataUpload()` - Manages uploaded CSV data and parsing state
- `useColumnMapping()` - Handles column-to-line mappings
- `useNameTagGeneration()` - Generates HTML/PDF from mapped data

**Key Pattern:** State flows through the wizard steps. Upload creates parsed data → Mapping creates column assignments → Preview generates the final output.

### API Architecture

**Two main server endpoints:**

1. **`/api/parse.post.ts`** - Accepts CSV file OR Google Sheets URL, returns `ParsedData`
   - Handles multipart form data (file upload) or JSON (Sheets URL)
   - Uses `server/utils/csv-parser.ts` for CSV parsing
   - Uses `server/utils/sheets-fetcher.ts` for Google Sheets integration

2. **`/api/generate.post.ts`** - Accepts CSV + column mapping, returns HTML or PDF
   - Takes `csvContent`, `mapping`, `hasHeaders`, `format` ('html' | 'pdf')
   - Uses `server/utils/column-mapper.ts` to map columns to tags
   - Uses `server/utils/html-generator.ts` to create HTML
   - Uses `server/utils/pdf-generator.ts` (Puppeteer) for PDF generation

### Server Utilities Pipeline

The generation pipeline flows through these utilities:

```
CSV Data
  → parseRawData() [data-parser.ts]
    → ParsedData { columns, headers, rowCount, preview }
  → parseCSVToPagesWithMapping() [column-mapper.ts]
    → NameTagPage[] { tags: NameTagRow[] }
  → generateNameTagsHTML() [html-generator.ts]
    → HTML string (TownStix US-10 format, 10 labels/page)
  → generatePDF() [pdf-generator.ts]
    → PDF Buffer (via Puppeteer)
```

**Critical:** `column-mapper.ts` handles the core business logic of mapping arbitrary CSV columns to the 3-line name tag format. It respects null mappings (empty lines).

### Component Architecture: Atomic Design

**Atoms** (`app/components/atoms/`):

- `Card.vue` - Glass card with backdrop blur
- `ContentBox.vue` - Inner content container
- `Button.vue`, `Badge.vue` - Basic interactive elements

**Molecules** (`app/components/molecules/`):

- `FeatureCard.vue` - Icon + title + description
- `FileUpload.vue` - Drag-and-drop file upload
- `ProgressIndicator.vue` - Wizard step progress
- `DataTable.vue` - Data preview table

**Organisms** (`app/components/organisms/`):

- `NameTagWizard.vue` - Main wizard container (orchestrates all steps)
- `ColumnMapper.vue` - Column mapping interface (select dropdowns for line1/2/3)
- `PreviewPanel.vue` - HTML preview with zoom controls
- `HeroSection.vue` - Landing page hero with features grid
- `AppHeader.vue` - Header with dark/light mode toggle

**Key Pattern:** Organisms compose molecules/atoms. The wizard state flows through props and composables.

### TypeScript Types

All core types are in `types/index.d.ts`:

- `NameTagRow` - Single tag with 3 lines (line1 is large/bold, line2/3 are smaller)
- `NameTagPage` - Array of up to 10 tags (TownStix US-10 = 10 labels per sheet)
- `ColumnMapping` - Maps each line to a column index (0-based) or null
- `ParsedData` - Parsed CSV with metadata (headers, column count, preview)
- `WizardStep` - Union type: 'upload' | 'mapping' | 'preview'
- `OutputFormat` - Union type: 'html' | 'pdf'

### CSS Architecture

**Component-scoped PostCSS** (not inline Tailwind):

```vue
<template>
  <div class="glass-card">
    <slot />
  </div>
</template>

<style lang="postcss" scoped>
.glass-card {
  @apply relative overflow-hidden rounded-xl border;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pacphi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
