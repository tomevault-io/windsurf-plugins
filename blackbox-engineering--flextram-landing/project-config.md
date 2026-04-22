---
trigger: always_on
description: - **Fork:** https://github.com/FlexTram/flextram_landing_updates
---

# FlexTram Landing Updates

## Repo origin
- **Fork:** https://github.com/FlexTram/flextram_landing_updates
- **Upstream (live site):** https://github.com/blackbox-engineering/flextram_landing
- **Live domain:** www.flextram.com (DNS points to blackbox-engineering.github.io)
- **Deployment:** Push to both remotes — changes go live on production immediately
  - `origin` = FlexTram/flextram_landing_updates (fork/backup)
  - `production` = blackbox-engineering/flextram_landing (live site)
- **Contributor access:** Direct push to production (no PRs needed)

## What's here
Static HTML site built on **Paper Kit 2 PRO v2.3.0** (Creative Tim). No build tools or package.json -- just HTML/CSS/JS served as-is.

### Pages (24 total)
- `index.html` -- Main FlexTram landing page (self-hosted video hero, client logos, contact form)
- `landing-page.html` -- Secondary landing page (full SEO meta)
- `system.html` -- System/product details page with before/after animation (easter egg, noindex)
- `solutions/index.html` -- Solutions hub with 17 industry cards (images + SEO descriptions)
- `solutions/airport-fbo.html` -- Airport & FBO use case page
- `solutions/resort-hotel.html` -- Resort & hotel shuttles
- `solutions/festivals-events.html` -- Festivals & outdoor events
- `solutions/stadiums-arenas.html` -- Stadiums & arenas
- `solutions/cruise-terminals.html` -- Cruise terminals
- `solutions/healthcare-hospitals.html` -- Healthcare & hospitals
- `solutions/university-campus.html` -- University & campus
- `solutions/warehouse-logistics.html` -- Warehouse & logistics
- `solutions/senior-living.html` -- Senior living communities
- `solutions/attractions-theme-parks.html` -- Attractions & theme parks
- `solutions/raceways-motorsports.html` -- Raceways & motorsports
- `solutions/air-shows.html` -- Air shows & aviation events
- `solutions/factory-tours.html` -- Factory & facility tours
- `solutions/convention-centers.html` -- Convention centers
- `solutions/labor-companies.html` -- Labor & staffing companies
- `solutions/golf-courses.html` -- Golf courses
- `solutions/planned-communities.html` -- Planned communities
- `blog/index.html` -- Blog hub page
- `blog/onsite-transportation-paradigm.html` -- First blog article (live)
- `blog/_drafts/` -- Scheduled articles (auto-published by GitHub Actions)

### Key directories
- `assets/css/` -- Stylesheets (Bootstrap purged, Paper Kit purged, `custom.css`, `global.css`, `solutions.css`, `use-case.css`, `blog.css`, `blog-post.css`)
- `assets/img/` -- Images (hero photos, event photos, logos, favicon)
- `assets/img/640/` -- Responsive 640px mobile variants (21 images)
- `assets/img/logos/` -- Client logos for scrolling carousel (26 logos)
- `assets/img/solutions/` -- Industry images for solutions cards (17 images)
- `assets/img/hero-*.jpg` -- Hero photos for all 17 solution sub-pages
- `assets/video/hero.mp4` -- Self-hosted hero video (4MB, 36s, 720p)
- `assets/js/` -- JavaScript (core only: jQuery, Popper, Bootstrap, Paper Kit, Slick)
- `_source_files/` -- Original source images (gitignored)
- `.github/workflows/` -- GitHub Actions (auto-publish blog drafts)

### Fonts
- **Homepage:** Inter (body) + Montserrat (headings) via Google Fonts
- **Solutions + Blog pages:** DM Sans (body) + Instrument Serif (headings) via Google Fonts

### SEO setup
- `sitemap.xml` -- All pages listed (solutions, blog, homepage)
- `robots.txt` -- Allow all, sitemap reference
- JSON-LD structured data: Organization, BreadcrumbList, BlogPosting, FAQPage schemas
- FAQPage JSON-LD on all 17 industry pages (86 FAQ items) + blog articles
- OG/Twitter meta tags on all pages
- Canonical URLs on all pages pointing to flextram.com
- Keyword-rich alt tags on every image across the site
- GA4 tracking (G-ZYK6B5M9QT) on all pages
- Instagram sameAs in Organization schema on all pages
- Registered in Google Search Console, sitemap submitted

### Performance optimizations
- CSS purged: Bootstrap 152KB→9KB, Paper Kit 313KB→34KB (422KB saved)
- Critical CSS inlined on homepage, stylesheets deferred
- Image lazy loading (46 images), fetchpriority="high" on hero images
- Responsive images: srcset with 640px mobile variants on 21 images
- Self-hosted video replaces YouTube iframe (554MB→4MB)
- 11 unused JS plugins removed from homepage and landing page
- Lighthouse: SEO 100, Best Practices 100, Performance 60, Accessibility 88

### Blog system
- Hub page at `/blog/` with article card grid
- Article template uses `blog-post.css` (based on use-case.css patterns)
- **Auto-publish workflow:** GitHub Actions runs daily at 9 AM UTC
  - Drafts in `blog/_drafts/` with filename `YYYY-MM-DD_slug.html`
  - When date arrives: moves to `blog/`, updates sitemap, adds card to hub page
  - Drafts include `BLOG_META` comment block for hub card metadata
  - Can also trigger manually from GitHub Actions tab
- **Scheduled articles:**
  - April 15: "The True Cost of the Golf Cart" (Operations)
  - April 22: "The Fan Experience Gap" (Fan Experience)
  - April 29: "Sponsorship's Untapped Frontier" (Revenue)

### Contact form
- Formspree endpoint: `https://formspree.io/f/mvzzoarr`
- Form ID: `#contact-form` (used as anchor target from all CTAs)
- All solution page and blog "Request Info" buttons link to `/#contact-form`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blackbox-engineering) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
