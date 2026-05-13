---
trigger: always_on
description: A modern, responsive web application for managing and tracking comic book collections with advanced analytics, financial tracking, and comprehensive organization features.
---

# Comic Book Collection Manager (comics.banast.as)

## Project Overview

A modern, responsive web application for managing and tracking comic book collections with advanced analytics, financial tracking, and comprehensive organization features.

**Live Site:** https://comics.banast.as

### Purpose
Digital comic book collection management tool for collectors to track their entire collection, analyze value and performance, search and filter comics, monitor financial gains/losses, and organize with custom tags and storage locations.

### Key Features
- 📚 Comprehensive comic tracking (title, series, issue, grade, value, storage, etc.)
- 📊 Advanced analytics dashboard with financial tracking
- 🔍 Smart search and filtering system
- 📱 Mobile-first responsive design
- 📈 Performance analytics (biggest gainers/losers)
- 🏷️ Custom tags and storage organization
- 🎨 Cover art display with lazy loading
- 📂 CSV import functionality
- 🔗 SEO-optimized with structured data
- 🚀 PWA-ready with manifest

## Tech Stack

- **Framework:** React 18.3+ with TypeScript 5.5+
- **Build Tool:** Vite 7.0+
- **Styling:** Tailwind CSS 3.4+
- **State Management:** Zustand 5.0+ (modern) + Custom React hooks (legacy)
- **Validation:** Zod 4.0+
- **Icons:** Lucide React
- **SEO:** react-helmet-async 2.0+
- **Node Version:** 18+

### Key Dependencies
- `react` & `react-dom` - Core React framework
- `zustand` - Lightweight state management
- `zod` - Schema validation
- `lucide-react` - Icon library
- `react-helmet-async` - Dynamic meta tags
- `tailwindcss` - Utility-first CSS framework

## Project Structure

```
comics.banast.as/
├── src/
│   ├── components/             # React components
│   │   ├── Dashboard.tsx       # Main statistics dashboard
│   │   ├── ComicCard.tsx       # Grid view card component
│   │   ├── ComicListView.tsx   # List view component
│   │   ├── ComicDetail.tsx     # Individual comic detail page
│   │   ├── ComicForm.tsx       # Add/edit comic form
│   │   ├── SeriesDetail.tsx    # Series-specific views
│   │   ├── CoverArtistDetail.tsx # Artist-specific views
│   │   ├── TagDetail.tsx       # Tag-specific views
│   │   ├── StorageLocationDetail.tsx # Storage views
│   │   ├── RawComicsDetail.tsx # Raw comics collection view
│   │   ├── SlabbedComicsDetail.tsx # Slabbed comics view
│   │   ├── VariantsDetail.tsx  # Variant covers view
│   │   ├── StorageLocationsListing.tsx # Storage overview
│   │   ├── FilterControls.tsx  # Search/filter UI
│   │   ├── SEO.tsx             # Dynamic SEO component
│   │   ├── ResponsiveImage.tsx # Optimized image loading
│   │   ├── LoadingSkeleton.tsx # Loading states
│   │   ├── ErrorBoundary.tsx   # Error handling
│   │   ├── FluidTypography.tsx # Responsive text
│   │   ├── TouchTarget.tsx     # Mobile touch optimization
│   │   ├── PerformanceMonitor.tsx # Performance tracking
│   │   └── lazyComponents.ts   # Lazy loading config
│   ├── stores/
│   │   └── comicStore.ts       # Zustand state store
│   ├── hooks/
│   │   ├── useComics.ts        # Legacy data management hook
│   │   ├── useComicFilters.ts  # Filtering and sorting logic
│   │   ├── useRouting.ts       # Navigation utilities
│   │   └── useResponsiveBreakpoint.ts # Responsive breakpoint hook
│   ├── types/
│   │   └── Comic.ts            # TypeScript interfaces
│   ├── utils/
│   │   ├── storage.ts          # Storage utilities
│   │   ├── performance.ts      # Performance utilities
│   │   └── routing.ts          # URL routing helpers
│   ├── validation/
│   │   └── comicSchema.ts      # Zod validation schemas
│   ├── data/
│   │   ├── comics.json         # Main collection data
│   │   └── test.json           # Test data
│   └── styles/
│       └── responsive.css      # Additional responsive styles
├── scripts/
│   └── generate-sitemap.js     # Sitemap generator
├── public/
│   ├── manifest.json           # PWA manifest
│   ├── robots.txt              # Crawler configuration
│   └── sitemap.xml             # Generated sitemap
├── index.html                  # Main HTML file with SEO meta tags
├── vite.config.ts              # Vite build configuration
├── tailwind.config.js          # Tailwind configuration
├── tsconfig.json               # TypeScript configuration
├── package.json                # Dependencies and scripts
├── example-comic-collection.json # Sample data (15 comics)
├── README.md                   # User documentation
└── SEO.md                      # SEO implementation guide
```

## Key Files

### Configuration Files
- **vite.config.ts** - Build optimization with code splitting, minification (Terser), manual chunks
- **tailwind.config.js** - Tailwind CSS configuration
- **tsconfig.json** - TypeScript strict mode configuration
- **postcss.config.js** - PostCSS with Autoprefixer
- **eslint.config.js** - ESLint rules

### Core Components
- **Dashboard.tsx** - Main analytics dashboard with statistics cards
- **ComicCard.tsx** - Individual comic card for grid view
- **ComicListView.tsx** - List view layout for comics
- **FilterControls.tsx** - Search, sort, and filter UI controls
- **SEO.tsx** - Dynamic SEO meta tags with react-helmet-async

### Detail Views (Lazy Loaded)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [banastas/comics.banast.as](https://github.com/banastas/comics.banast.as) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
