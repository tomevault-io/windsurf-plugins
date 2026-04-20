---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start development server:**
```bash
pnpm dev
```
Launches Next.js with Turbopack at http://localhost:3000 with hot reload enabled.

**Production build:**
```bash
pnpm build
```
Creates optimized production bundle. Run before committing significant changes.

**Serve production build locally:**
```bash
pnpm start
```
Use to validate production behavior and deployment fixes.

**Lint code:**
```bash
pnpm lint
```
Runs ESLint with Next.js and TypeScript rules. Must pass before opening PRs.

## Architecture Overview

### Tech Stack
- **Framework:** Next.js 15.5.2 (React 19.1.0) with App Router
- **CMS:** Sanity v4.9.0 (headless CMS for content management)
- **Styling:** Tailwind CSS v4 with semantic tokens
- **Animation:** Motion v12.23.12 (motion/react)
- **Forms:** react-hook-form v7.62.0 + Zod v4.1.5 validation
- **Progress Indicators:** nextjs-toploader v3.9.17
- **Icons:** @sanity/icons v3.7.4 + react-icons v5.5.0
- **Utilities:** clsx v2.1.1 + tailwind-merge v3.3.1
- **Content Rendering:** @portabletext/react v4.0.3
- **Language:** TypeScript v5 (strict mode)
- **Deployment:** Cloudflare Pages (@opennextjs/cloudflare v1.9.0)

### Directory Structure
```
src/
├── app/                  # Next.js App Router
│   ├── (main)/          # Main site route group
│   │   ├── aboutus/     # About Us page
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── blogs/       # Blog listing & posts
│   │   │   ├── [slug]/
│   │   │   │   ├── BlogPostClient.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── sitemap.ts
│   │   ├── media/       # Media gallery
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── products/    # Products section
│   │   │   ├── [slug]/
│   │   │   │   ├── [city]/      # Location-based product pages
│   │   │   │   │   └── page.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── sitemap.ts
│   │   ├── services/    # Services section
│   │   │   ├── [slug]/
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── layout.tsx   # Main site layout (Navbar + Footer)
│   │   └── page.tsx     # Homepage
│   ├── studio/          # Sanity Studio admin interface
│   │   └── [[...tool]]/page.tsx
│   ├── layout.tsx       # Root layout with fonts & metadata
│   ├── globals.css      # Global styles & Tailwind config
│   ├── not-found.tsx    # 404 page
│   ├── robots.ts        # robots.txt generation
│   └── sitemap.ts       # XML sitemap generation
├── components/          # Reusable React components (PascalCase)
│   ├── aboutus/
│   │   ├── CertificateModal.tsx
│   │   ├── CertificatesSection.tsx
│   │   ├── TeamMemberModal.tsx
│   │   ├── TeamSection.tsx
│   │   ├── VisionMissionValues.tsx
│   │   └── WhyUsSection.tsx
│   ├── blog/
│   │   ├── BlogCard.tsx
│   │   ├── FeaturedBlogCard.tsx
│   │   └── PortableTextRenderer.tsx
│   ├── homepage/
│   │   ├── AboutUs.tsx
│   │   ├── BlogSection.tsx
│   │   ├── ClientsMarquee.tsx
│   │   ├── ContactSection.tsx
│   │   ├── FAQSection.tsx
│   │   ├── Hero.tsx
│   │   ├── MediaPreview.tsx
│   │   ├── SEOContentSection.tsx
│   │   ├── Services.tsx
│   │   └── Testimonials.tsx
│   ├── layout/
│   │   ├── Footer.tsx
│   │   ├── FooterSitemapLinks.tsx
│   │   └── Navbar.tsx
│   ├── media/
│   │   ├── MediaCard.tsx
│   │   ├── MediaFilters.tsx
│   │   ├── MediaPageClient.tsx
│   │   └── MediaPreview.tsx
│   ├── products/
│   │   ├── ProductCard.tsx
│   │   ├── ProductCarousel.tsx
│   │   ├── ProductGalleryModal.tsx
│   │   └── ProductsInteractive.tsx
│   ├── services/
│   │   ├── ServiceCard.tsx
│   │   └── ServicesGrid.tsx
│   └── [Shared Components]
│       ├── AnimatedNumbers.tsx
│       ├── Breadcrumb.tsx
│       ├── CallToActionSection.tsx
│       ├── CatalogModal.tsx
│       ├── DownloadCatalogButton.tsx
│       ├── FAQ.tsx
│       ├── Features.tsx
│       ├── Quote.tsx
│       ├── QuoteCTA.tsx
│       └── WhatsAppButton.tsx
├── hooks/               # Custom React hooks
│   ├── useSmoothScroll.ts     # Smooth scrolling navigation
│   └── useViewTransition.ts   # View transitions API
└── sanity/              # Sanity CMS integration
    ├── lib/             # Type definitions & client config
    │   ├── aboutTypes.ts
    │   ├── blogTypes.ts
    │   ├── certificateTypes.ts
    │   ├── client.ts           # Sanity client configuration
    │   ├── clientTypes.ts
    │   ├── image.ts            # Image URL builder
    │   ├── live.ts             # Live preview support
    │   ├── mediaTypes.ts
    │   ├── productTypes.ts
    │   ├── serviceTypes.ts
    │   └── testimonialTypes.ts
    ├── schemaTypes/     # Content type schemas (26 schemas)
    │   ├── authorType.ts
    │   ├── blockContentType.ts
    │   ├── categoryType.ts
    │   ├── certificateType.ts
    │   ├── clients.ts
    │   ├── companyInfo.ts
    │   ├── contactInfo.ts
    │   ├── faq.ts
    │   ├── homePageSeo.ts
    │   ├── homePageSettings.ts
    │   ├── keyFeatures.ts
    │   ├── media.ts
    │   ├── postType.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dev-rocketdevelopers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
