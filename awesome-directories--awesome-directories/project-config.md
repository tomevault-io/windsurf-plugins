---
trigger: always_on
description: Manages authentication state:
---

# Claude Context: Awesome Directories

## Project Overview

**Awesome Directories** is a curated directory aggregator web application that helps indie hackers, bootstrappers, and solopreneurs discover high-quality product launch directories for their SaaS products. The site aggregates 300+ verified directories with advanced filtering capabilities.

- **Live Site**: https://awesome-directories.com
- **Repository**: awesome-directories/awesome-directories
- **License**: Apache 2.0
- **Hosting**: GitHub Pages (static SSG)
- **Architecture**: Astro.js Static Site Generation with Vue.js islands for interactivity

### Core Value Proposition

- Curated, verified directories updated weekly
- Advanced filtering by Domain Rating (DR), category, pricing, and link type
- User authentication with favorites and project-based submission tracking
- Directory detail pages with related directories, ratings, and reviews
- User directory submission system with review workflow and email notifications
- Project management for tracking submissions across multiple products
- Automated SEO metrics updates via Ahrefs API and Supabase Edge Functions
- Comprehensive statistics page with interactive charts
- Blog with SEO-optimized content, tags, search, and RSS feed

## Technology Stack

### Frontend Framework

- **Astro.js 5.16.0** (Static Site Generation with Content Collections)
- **Vue.js 3** (Composition API for interactive islands/components)
- **Vite 7.2.2** (build tool, dev server on port 3000)
- **Tailwind CSS 4.1.17** (utility-first CSS with PostCSS)
- **TypeScript-ready** (ES modules)

**Important**: The project migrated from a Vue.js SPA to Astro.js SSG for better SEO and performance. Vue components are used for interactive features via Astro's component islands. Astro Content Collections power the blog system.

### Key Libraries

- `@supabase/supabase-js` ^2.83.0 - Database client (build-time and runtime)
- `@astrojs/rss` ^4.0.13 - RSS feed generation for blog
- `@astrojs/mdx` ^4.3.11 - MDX support for enhanced blog posts (dev dependency)
- `chart.js` ^4.5.1 - Interactive charts for statistics page
- `pagefind` ^1.4.0 - Static search indexing for blog (dev dependency)
- `html2canvas` ^1.4.1 + `jspdf` ^2.5.2 - PDF export functionality
- `papaparse` ^5.5.3 - CSV parsing/export
- `slugify` ^1.6.6 - URL slug generation
- `nanostores` ^0.10.3 - Lightweight state management
- `uhtml` ^5.0.9 - Lightweight DOM rendering
- `ky` ^1.14.0 - HTTP client
- `loglevel` ^1.9.2 - Logging utility
- `prettier` ^3.6.2 - Code formatting

### Backend & Services

- **Supabase** - PostgreSQL + Auth + Realtime
- **PostgreSQL** with Row Level Security (RLS)
- **Supabase Edge Functions** (Deno runtime)
  - `update-seo-data` - Updates Ahrefs metrics via pg_cron scheduled jobs
  - `send-approval-email` - Sends approval emails via Resend when directories are approved
  - `send-rejection-email` - Sends rejection emails with feedback
  - `send-welcome-email` - Sends welcome emails on user signup
  - `send-submission-confirmation` - Confirms directory submission receipt
  - `send-admin-notification` - Notifies admins of new submissions
- **Resend** - Transactional email service for all notifications
- **Listmonk** - Self-hosted newsletter service (newsletter.meysam.io)
- **Pirsch** - Privacy-first analytics - _optional_
- **Ahrefs API** - SEO metrics (DR, traffic estimates)
- **Giscus** - GitHub Discussions for comments - _optional_

### Package Management

- **Bun** (primary - faster than npm/yarn)
- **npm** also supported (package-lock.json present)

### Deployment & CI/CD

- **GitHub Pages** (production hosting)
- **GitHub Actions** (build and deploy pipeline)
- **Netlify** (PR preview deployments)

## Project Structure

```
/
├── .github/workflows/
│   └── ci.yml                      # CI/CD pipeline
├── public/                         # Static assets
│   ├── data/                       # Generated at build time
│   │   ├── directories.json        # All directories data
│   │   └── stats.json              # Statistics data for charts
│   ├── pagefind/                   # Search index (generated)
│   └── robots.txt                  # SEO robots file
├── src/
│   ├── pages/                      # Astro pages (file-based routing)
│   │   ├── index.astro            # Home page (directory listing)
│   │   ├── about.astro            # About page
│   │   ├── stats.astro            # Statistics page with charts
│   │   ├── terms.astro            # Terms of Service
│   │   ├── privacy.astro          # Privacy Policy
│   │   ├── favorites.astro        # User favorites page (auth required)
│   │   ├── submissions.astro      # User submissions tracker (auth required)
│   │   ├── my-submissions.astro   # User's directory submissions status (auth required)
│   │   ├── submit.astro           # Submit new directory form (auth required)
│   │   ├── 404.astro              # 404 error page
│   │   ├── directory/
│   │   │   └── [slug].astro       # Dynamic directory detail pages
│   │   └── blog/
│   │       ├── index.astro        # Blog listing page
│   │       ├── [slug].astro       # Individual blog post pages
│   │       ├── [...page].astro    # Blog pagination
│   │       └── tags/
│   │           ├── [tag].astro    # Posts by tag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awesome-directories/awesome-directories](https://github.com/awesome-directories/awesome-directories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
