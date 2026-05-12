---
trigger: always_on
description: This file is the single source of truth for Codex when working in this repository. Every code generation, refactor, or review **must** follow these instructions exactly.
---

# AGENTS.md — Master Instruction

This file is the single source of truth for Codex when working in this repository. Every code generation, refactor, or review **must** follow these instructions exactly.

## Project Identity

**Silo** — a Chrome extension that audits the SEO health of any web page during local development. Built for developers who want Lighthouse-grade SEO feedback in a persistent side panel while they build.

**Tech stack**: React 19, TypeScript, Vite 7, Tailwind CSS 4, CRXJS Vite Plugin, Manifest V3.

## Baseline Standard

All audit logic **must** follow the standards defined by Google Lighthouse SEO Audits and the referenced accessibility rules. The Master Audit Registry below is the **absolute reference** — every validator's `sourceName` and `sourceUrl` must match it exactly. **Any deviation from these sources is not allowed unless explicitly requested.**

**Every `ValidationItem` must include source attribution:**

```ts
interface ValidationItem {
  severity: Severity          // 'ok' | 'warning' | 'error' | 'critical'
  message: string             // What is wrong (or right)
  advice: string              // Why it matters
  fix: string                 // How to fix it
  sourceName: string          // Must match the registry below
  sourceUrl: string           // Must match the registry below
}
```

## Master Audit Registry

This is the official mapping. When building or updating any validator, use the `sourceName` and `sourceUrl` from this table verbatim.

- [x] **1. `crawlability`** — Page isn't blocked from indexing · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/is-crawlable`
- [x] **2. `title`** — Document has a `<title>` element · Deque University · `https://dequeuniversity.com/rules/axe/4.11/document-title`
- [x] **3. `description`** — Document has a meta description · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/meta-description`
- [ ] **4. `http-status`** — Page has successful HTTP status code (200 OK) · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/http-status-code/` *(skipped — not actionable in extension context; page is always loaded)*
- [x] **5. `link-text`** — Links have descriptive text (avoid "click here") · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/link-text/`
- [x] **6. `crawlable-links`** — Links must use proper `href` attributes · Google Search · `https://developers.google.com/search/docs/crawling-indexing/links-crawlable`
- [x] **7. `robots-txt`** — `robots.txt` is valid and well-formed · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/invalid-robots-txt/`
- [x] **8. `alt-text`** — Image elements have `[alt]` attributes · Deque University · `https://dequeuniversity.com/rules/axe/4.11/image-alt`
- [x] **9. `hreflang`** — Document has a valid `hreflang` for multi-language · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/hreflang/`
- [x] **10. `canonical`** — Document has a valid `rel=canonical` · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/canonical/`
- [x] **11. `viewport`** — Page has a `<meta name="viewport">` tag with `width=device-width` · Lighthouse · `https://developer.chrome.com/docs/lighthouse/pwa/viewport/`
- [x] **12. `structured-data`** — JSON-LD structured data is valid and well-formed · Google Search Central · `https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data`
- [x] **13. `sitemap`** — `sitemap.xml` is valid and well-formed · Google Search Central · `https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview`

## Audit Roadmap

### Phase 1 — Lighthouse Compliance (The 10 Core Audits)

Implement all 10 audits from the Master Audit Registry above. Each must return `ValidationItem` objects with the exact `sourceName` and `sourceUrl` from the registry.

- [x] `title` — Done
- [x] `description` — Done
- [x] `canonical` — Done
- [x] `alt-text` — Done (retrofitted with source attribution)
- [x] `crawlable-links` — Done (retrofitted with source attribution)
- [x] `link-text` — Done
- [x] `crawlability` — Done (meta robots + X-Robots-Tag header via background service worker)
- [ ] `http-status` — Skipped (not actionable in extension context; page is already loaded)
- [x] `robots-txt` — Done (fetched from side panel)
- [x] `hreflang` — Done

### Phase 2 — Developer Utilities ✅

- [x] Real-time Broken Link Checker
- [x] Localhost Absolute Link Detector
- [x] Heading Tree Visualizer

### Phase 3 — Content Intelligence ✅

- [x] Flesch Reading Ease scoring
- [x] One-click copy fix in AuditAdvice
- [x] Scan diff / regression tracking
- [x] Editable social preview + OG code generator
- [x] Keyword analysis

### Phase 4 — AI-Powered Features ✅

- [x] API key management + settings UI
- [x] AI meta title/description generator
- [x] AI alt text generator (Codex vision)
- [x] AI structured data (JSON-LD) generator
- [x] Content ↔ meta relevance score
- [x] Tone analysis

### Phase 5 — Power User Features ✅

- [x] SEO history timeline (SVG sparkline)
- [x] HTML export report

## Architecture

**Extension entry points** (defined in `manifest.config.ts`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nauvalazhar/silo](https://github.com/nauvalazhar/silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
