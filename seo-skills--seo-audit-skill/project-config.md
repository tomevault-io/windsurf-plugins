---
trigger: always_on
description: Audit websites for SEO, technical, content, security, JS rendering, and AI readiness using SEOmator CLI. Returns LLM-optimized reports with health scores across 251 rules and 20 categories. Use when analyzing websites, debugging SEO issues, or checking site health.
---


# SEO Audit Skill

Audit websites for SEO, technical, content, performance, security, JavaScript rendering, and AI readiness using the SEOmator CLI.

SEOmator provides comprehensive website auditing by analyzing website structure and content against **251 rules** across **20 categories**.

It provides a list of issues with severity levels, affected URLs, and actionable fix suggestions.

## Links

* SEOmator npm package: [npmjs.com/package/@seomator/seo-audit](https://www.npmjs.com/package/@seomator/seo-audit)
* GitHub repository: [github.com/seo-skills/seo-audit-skill](https://github.com/seo-skills/seo-audit-skill)
* Web UI: [seomator.com/free-seo-audit-tool](https://seomator.com/free-seo-audit-tool)

## What This Skill Does

This skill enables AI agents to audit websites for **251 rules** in **20 categories**, including:

- **Core SEO** (19 rules): Canonical URLs, indexing directives, title uniqueness, canonical conflicts/loops
- **Performance** (22 rules): LCP, CLS, FCP, TTFB, INP, compression, caching, minification, HTTP/2
- **Links** (19 rules): Broken links, redirect chains, anchor text, orphan pages, localhost/fragment links
- **Images** (14 rules): Alt text, dimensions, lazy loading, modern formats, alt length, background images
- **Security** (16 rules): HTTPS, HSTS, CSP, external link safety, leaked secrets, SSL expiry/protocol
- **Technical SEO** (13 rules): robots.txt, sitemap.xml, URL structure, 404 pages, soft 404s, error codes
- **Crawlability** (18 rules): Sitemap conflicts, indexability signals, canonical chains, pagination issues
- **Structured Data** (13 rules): Schema.org markup, Article, Organization, FAQ, Product, Breadcrumb
- **JavaScript Rendering** (13 rules): Rendered DOM checks, raw vs rendered mismatches, SSR detection
- **Accessibility** (12 rules): ARIA labels, color contrast, form labels, landmarks, touch targets
- **Content** (17 rules): Word count, readability, keyword density, duplicate detection, pixel widths
- **Social** (9 rules): Open Graph tags, Twitter cards, share buttons, profile links
- **E-E-A-T** (14 rules): Author bylines, citations, trust signals, about/contact pages, YMYL detection
- **URL Structure** (14 rules): Keyword slugs, stop words, uppercase, underscores, session IDs, tracking params
- **Redirects** (8 rules): Redirect loops, types (301/302), meta refresh, JavaScript redirects, broken redirects
- **Mobile** (5 rules): Font sizes, horizontal scroll, intrusive interstitials, viewport issues
- **Internationalization** (10 rules): lang attribute, hreflang validation (return links, conflicts, mismatches)
- **HTML Validation** (9 rules): Doctype, charset, head structure, lorem ipsum, multiple titles/descriptions
- **AI/GEO Readiness** (5 rules): Semantic HTML, AI bot access, llms.txt, schema drift
- **Legal Compliance** (1 rule): Cookie consent

The audit crawls the website, analyzes each page against audit rules, and returns a comprehensive report with:
- Overall health score (0-100) with letter grade (A-F)
- Category breakdowns with pass/warn/fail counts
- Specific issues with affected URLs grouped by rule
- Actionable fix recommendations

## When to Use

Use this skill when you need to:
- Analyze a website's SEO health
- Debug technical SEO issues
- Check for broken links and redirect chains
- Validate meta tags, canonical URLs, and structured data
- Audit security headers, SSL, and HTTPS
- Check accessibility compliance
- Analyze JavaScript rendering and SSR compatibility
- Evaluate AI/GEO readiness (semantic HTML, llms.txt, bot access)
- Detect duplicate content across pages
- Validate hreflang and internationalization setup
- Check HTML document structure and validation
- Generate site audit reports in multiple formats
- Compare site health before/after changes

## Prerequisites

This skill requires the SEOmator CLI to be installed.

### Installation

```bash
npm install -g @seomator/seo-audit
```

### Verify Installation

Check that seomator is installed and the system is ready:

```bash
seomator self doctor
```

This checks:
- Node.js version (18+ recommended)
- npm availability
- Chrome/Chromium for Core Web Vitals and JS rendering
- Write permissions for ~/.seomator
- Local config file presence

## Setup

Running `seomator init` creates a `seomator.toml` config file in the current directory.

```bash
seomator init                    # Interactive setup
seomator init -y                 # Use defaults
seomator init --preset blog      # Blog-optimized config
seomator init --preset ecommerce # E-commerce config
seomator init --preset ci        # Minimal CI config
```

If there is no `seomator.toml` in the directory, CREATE ONE with `seomator init` before running audits.

## Usage

### AI Agent Best Practices

**YOU SHOULD always prefer `--format llm`** - it provides token-optimized XML output specifically designed for AI agents (50-70% smaller than JSON).

When auditing:
1. **Prefer live websites** over local dev servers for accurate performance and rendering data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seo-skills/seo-audit-skill](https://github.com/seo-skills/seo-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
