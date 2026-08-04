---
trigger: always_on
description: This document provides guidance for AI assistants working with the `@shopware/cms-base-layer` package.
---

# AI Agent Guide for @shopware/cms-base-layer

This document provides guidance for AI assistants working with the `@shopware/cms-base-layer` package.

## TL;DR

**What**: Nuxt layer providing Vue components for Shopware 6 Shopping Experiences (CMS)
**Purpose**: Pre-built, customizable CMS components
**Tech**: Vue 3, Nuxt 4, @shopware/composables, @shopware/helpers

**Key Concepts**:

- CMS Page → Sections → Blocks → Elements (hierarchical structure)
- Components auto-registered globally via Nuxt layer
- Customizable via app.config.ts and component overriding

**Quick Start**:

```bash
pnpm run build  # Build the package
pnpm run dev    # Stub mode for development
pnpm run test   # Run tests
```

## Package Overview

The `@shopware/cms-base-layer` is a Nuxt layer that provides a complete implementation of Shopware's Shopping Experiences CMS system. It renders CMS pages created in Shopware Administration using Vue components.

## Architecture

### CMS Hierarchy

Shopware CMS follows a strict hierarchical structure:

```
CmsPage
├── CmsSection (layout container)
│   ├── CmsBlock (content grouping)
│   │   ├── CmsElement (actual content)
│   │   └── CmsElement
│   └── CmsBlock
└── CmsSection
```

### Component Naming Convention

Components follow a predictable naming pattern based on the CMS structure:

| Level   | Pattern            | Example                                          |
| ------- | ------------------ | ------------------------------------------------ |
| Page    | `CmsPage`          | `CmsPage.vue`                                    |
| Section | `CmsSection{Type}` | `CmsSectionDefault.vue`, `CmsSectionSidebar.vue` |
| Block   | `CmsBlock{Type}`   | `CmsBlockImage.vue`, `CmsBlockText.vue`          |
| Element | `CmsElement{Type}` | `CmsElementImage.vue`, `CmsElementText.vue`      |

## Directory Structure

```
cms-base-layer/
├── app/
│   ├── app.config.ts              # Default app configuration
│   ├── assets/                    # Static assets (icons, images)
│   ├── components/
│   │   ├── Sw*.vue               # Shared components (SwProductCard, etc.)
│   │   ├── listing-filters/      # Product listing filter components
│   │   ├── public/
│   │   │   └── cms/              # CMS components (auto-registered)
│   │   │       ├── CmsPage.vue
│   │   │       ├── CmsGenericBlock.vue
│   │   │       ├── CmsGenericElement.vue
│   │   │       ├── block/        # Block components
│   │   │       ├── element/      # Element components
│   │   │       ├── section/      # Section components
│   │   │       └── skeleton/     # Loading skeleton components
│   │   └── ui/                   # Base UI components (prefixed with Sw)
│   ├── composables/              # Layer-specific composables
│   │   └── useLcpImagePreload.ts # LCP image preload via <link rel="preload">
│   ├── helpers/                  # Utility functions
│   │   └── cms/getImageSizes.ts  # Slot count → responsive sizes mapping
│   ├── plugins/                  # Nuxt plugins
│   └── providers/                # Image providers (Shopware)
├── nuxt.config.ts                # Layer configuration
└── package.json
```

## Key Files

### Entry Points

- [nuxt.config.ts](nuxt.config.ts) - Layer configuration, component registration, image presets
- [app/app.config.ts](app/app.config.ts) - Runtime configuration defaults

### Core CMS Components

- [app/components/public/cms/CmsPage.vue](app/components/public/cms/CmsPage.vue) - Main entry point for rendering CMS pages
- [app/components/public/cms/CmsGenericBlock.vue](app/components/public/cms/CmsGenericBlock.vue) - Generic block renderer (provides responsive image sizes via inject)
- [app/components/public/cms/CmsGenericElement.vue](app/components/public/cms/CmsGenericElement.vue) - Generic element renderer
- [app/composables/useLcpImagePreload.ts](app/composables/useLcpImagePreload.ts) - Preloads LCP image during SSR
- [app/helpers/cms/getImageSizes.ts](app/helpers/cms/getImageSizes.ts) - Maps block slot count to responsive `sizes` attribute

### Sections (Layout)

| Component           | Purpose                     |
| ------------------- | --------------------------- |
| `CmsSectionDefault` | Full-width section          |
| `CmsSectionSidebar` | Section with sidebar layout |

### Common Blocks

| Component                    | Purpose                                               |
| ---------------------------- | ----------------------------------------------------- |
| `CmsBlockImage`              | Single image display                                  |
| `CmsBlockText`               | Text content                                          |
| `CmsBlockImageText`          | Image with text side-by-side                          |
| `CmsBlockProductListing`     | Product grid/list                                     |
| `CmsBlockProductSlider`      | Product carousel                                      |
| `CmsBlockImageSlider`        | Image carousel                                        |
| `CmsBlockImageGallery`       | Image gallery grid                                    |
| `CmsBlockForm`               | Contact/newsletter forms                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shopware/frontends](https://github.com/shopware/frontends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
