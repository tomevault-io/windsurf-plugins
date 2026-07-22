---
trigger: always_on
description: > This file is auto-generated. Do not edit directly.
---

# Nuxt SEO Best Practices - Complete Reference

> This file is auto-generated. Do not edit directly.
> Edit individual rule files in the `rules/` directory and run `bun run build`.

# Nuxt SEO Best Practices

Comprehensive SEO optimization guide for Nuxt 4 applications deployed to Cloudflare Pages/Workers. Contains 11 rules across 4 categories, prioritized by impact to guide automated refactoring and code generation.

## When to Apply

Reference these guidelines when:

- Generating dynamic OG images on Cloudflare Workers
- Setting up page-level SEO meta tags and composables
- Adding JSON-LD structured data
- Configuring Nitro for Cloudflare Pages deployment
- Handling SSR-incompatible client-only libraries
- Setting up social sharing meta (Open Graph, Twitter Cards)
- Working with Satori for image generation (NEVER use React)

## Rule Categories by Priority

| Priority | Category                  | Impact   | Prefix    |
| -------- | ------------------------- | -------- | --------- |
| 1        | OG Image Generation       | CRITICAL | `og-`     |
| 2        | Page SEO & Meta           | HIGH     | `meta-`   |
| 3        | Structured Data           | MEDIUM   | `schema-` |
| 4        | Cloudflare & Nitro Config | HIGH     | `cf-`     |

## Quick Reference

### 1. OG Image Generation (CRITICAL)

- `og-cf-workers` - Generate dynamic OG images on Cloudflare Workers with @cf-wasm/og
- `og-no-react` - Use plain JS objects for Satori elements, NEVER React
- `og-cache-headers` - Cache OG images with immutable headers for CDN

### 2. Page SEO & Meta (HIGH)

- `meta-use-page-seo` - Reusable composable for consistent page-level SEO
- `meta-social-tags` - Proper Open Graph and Twitter Card meta tags
- `meta-canonical-url` - Canonical URLs and og:url for every page

### 3. Structured Data (MEDIUM)

- `schema-json-ld` - JSON-LD structured data in app.vue for Google Rich Results

### 4. Cloudflare & Nitro Config (HIGH)

- `cf-compatibility-date` - Pin compatibilityDate, never use 'latest'
- `cf-nitro-config` - Nitro config for CF Pages (nodeCompat, process.stdout, WASM)
- `cf-ssr-externals` - SSR external config for client-only libraries
- `cf-wasm-import` - WASM module configuration for Cloudflare Workers

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/og-cf-workers.md
rules/meta-use-page-seo.md
rules/_sections.md
```

Each rule file contains:

- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and Nuxt/Cloudflare-specific notes

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`

---

# Detailed Rules

### Pin compatibilityDate, Never Use 'latest'

**Impact:** HIGH - Prevents unpredictable Nitro behavior changes between builds

## Pin compatibilityDate, Never Use 'latest'

Nuxt's `compatibilityDate` controls which Nitro runtime behaviors are active. Using `'latest'` resolves to a different date on every build, which can silently change how your app behaves in production.

**Incorrect (using 'latest'):**

```typescript
// ❌ WRONG — 'latest' resolves to a different date on each build
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: 'latest',
  // Today it might be 2025-07-18, tomorrow 2025-07-19
  // Each date can change Nitro's internal behavior
});
```

**Correct (pinned date):**

```typescript
// ✅ CORRECT — Pinned to a specific date
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: '2025-07-18',
  // Consistent behavior across all builds
  // Update intentionally when you want new Nitro features
});
```

**When to update:**

- When upgrading Nuxt or Nitro versions
- When you need a specific Nitro feature that requires a newer date
- Always test after updating — some changes are breaking

**How to find the right date:**

- Use the date of your current Nuxt release
- Check [Nitro changelog](https://github.com/unjs/nitro/releases) for what changed
- Pin to the latest date that works with your deployment target

**Real-world impact:** A `compatibilityDate` change can affect:

- How `process.env` is handled in server routes
- WASM module loading behavior
- Node.js API compatibility layer
- Response header defaults

---

### Configure Nitro for Cloudflare Pages Deployment

**Impact:** HIGH - Ensures WASM support, Node.js compatibility, and proper builds on CF Pages

## Configure Nitro for Cloudflare Pages Deployment

Cloudflare Pages/Workers have specific requirements for WASM modules, Node.js APIs, and global objects. Without proper Nitro configuration, builds fail or runtime errors occur.

**Incorrect (minimal config):**

```typescript
// ❌ WRONG — Missing critical CF Workers config
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages',
  },
});
```

**Correct (full CF Pages config):**

```typescript
// ✅ CORRECT — Full Cloudflare Pages configuration
export default defineNuxtConfig({
  compatibilityDate: '2025-07-18',

  nitro: {
    preset: 'cloudflare-pages',
  },

  cloudflare: {
    // Enable Node.js API compatibility (Buffer, crypto, etc.)
    nodeCompat: true,
  },

  vite: {
    // Replace process.stdout (not available in CF Workers)
    define: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geoql/v-maplibre](https://github.com/geoql/v-maplibre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
