---
trigger: always_on
description: >
---


# Vercel Cost Guard

Audit a Next.js/Vercel project for patterns that cause high hosting costs — bandwidth, compute, and invocation charges.

Vercel's pay-per-use pricing means code patterns directly affect your hosting bill. A missing `preload="none"`, an unscoped middleware, or a single `cookies()` call can silently multiply costs. This audit catches these patterns before they show up on an invoice.

## How to Run the Audit

Perform the following 22 checks in order using your built-in tools (Glob, Grep, Read). After completing all checks, compile findings into the report format at the bottom.

**Report generation rules:**
- For every CRITICAL finding, include a copy-paste-ready fix (shell command, code snippet, or config change) drawn from reference/optimization-guide.md. Substitute the project's actual file paths into the commands.
- Estimate dollar impact where possible. Use the cost formulas from reference/vercel-pricing.md. Assume 50K monthly visitors as a baseline unless the project's traffic is known. State your assumption.
- Track positive findings ("GOOD" patterns from Checks 2, 8, and 11) — these are used in the report when the project has no critical or warning issues.

**Important:** Search `.ts` and `.js` files in addition to `.tsx` and `.jsx` — blog content and HTML strings are often defined in TypeScript template literals, not just JSX files.

---

## Check 1: Public Directory Assets

Use Glob to find all files in `public/`. For each media file, note its size.

**CRITICAL — Files > 10 MB:**
Any file over 10 MB in `public/` will cause significant bandwidth costs under traffic. Each visitor downloads it fully.
- Fix: Compress the file, convert to a smaller format, or move to an external CDN (Cloudflare R2, AWS S3).

**WARNING — Files 5-10 MB:**
Large but not extreme. Should be compressed before any launch or promotion.

**CRITICAL — Uncompressed formats:**
Flag any files with these extensions — they have compressed alternatives that are 2-20x smaller:
- `.wav`, `.flac` → convert to `.mp3` or `.m4a`
- `.mov`, `.avi` → convert to `.mp4` (H.264)
- `.bmp`, `.tiff`, `.tif`, `.raw`, `.psd` → convert to `.webp` or `.avif`

Calculate the total size of all media files in `public/` and list the top 5 largest files.

For detailed compression commands, see [reference/optimization-guide.md](reference/optimization-guide.md).

---

## Check 2: Media Preload Attributes

Use Grep to search all `.tsx`, `.jsx`, `.ts`, `.js`, and `.html` files (excluding `node_modules/` and `.next/`) for `<audio` and `<video` elements.

**CRITICAL — `preload="auto"`:**
Pattern: `<(audio|video)[^>]*preload\s*=\s*["']auto["']`
The browser downloads the ENTIRE file on page load. This is the most expensive setting.
- Fix: Change to `preload="none"`.

**WARNING — `preload="metadata"`:**
Pattern: `<(audio|video)[^>]*preload\s*=\s*["']metadata["']`
The browser downloads file headers to read duration/codec info. For large files, this can be several MB per element.
- Fix: Change to `preload="none"` for cost safety.

**CRITICAL — Missing preload attribute:**
Find `<audio` and `<video` tags that do NOT have a `preload` attribute. Many browsers default to `preload="auto"` when the attribute is missing, which downloads the entire file.
- Fix: Add `preload="none"` explicitly.

**GOOD — `preload="none"`:**
Note any elements correctly using `preload="none"` as positive findings.

---

## Check 3: Next.js Config

Use Glob to find `next.config.ts`, `next.config.mjs`, or `next.config.js`. Read the file.

**WARNING — Missing `images.minimumCacheTTL` (Next.js 14/15):**
Without this setting, optimized images expire quickly and must be re-optimized on the next request (costs compute + bandwidth). Next.js 16+ increased the default from 60s to 14,400s (4 hours), so this is less critical on v16+ but still important on v14/v15 where the default is only 60s.
- Fix: Add `minimumCacheTTL: 2592000` (30 days) to the `images` config.

**WARNING — Low cache TTL (< 86400 seconds / 1 day):**
Even with the setting present, a value under 1 day causes frequent re-optimization.
- Fix: Increase to at least `2592000` (30 days) for static images.

**INFO — Missing AVIF format:**
If `images.formats` does not include `'image/avif'`, images are 20-50% larger than they could be.
- Fix: Add `formats: ['image/avif', 'image/webp']` to images config.

**WARNING — Missing custom Cache-Control headers:**
Check if the config has a `headers()` function with `Cache-Control` for static assets. Without custom headers, static assets may not be cached effectively at the CDN edge.
- Fix: Add long-lived cache headers for static file extensions. See [reference/optimization-guide.md](reference/optimization-guide.md).

**WARNING — Overly broad `remotePatterns`:**
Check the `images.remotePatterns` array. Flag if any entry uses `hostname: '**'` or a very permissive wildcard (e.g., `hostname: '*.com'`). Broad remote patterns allow any external source to trigger image optimization (billed per transformation — see [reference/vercel-pricing.md](reference/vercel-pricing.md)). Wildcard hostnames also create SSRF/DoS risk against the image optimizer (CVE-2025-59471).
- Fix: Restrict `remotePatterns` to specific, known hostnames your app actually uses.

---

## Check 4: Image Optimization

**WARNING — Raw `<img>` tags instead of next/image:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jshchnz/vercel-cost-guard](https://github.com/jshchnz/vercel-cost-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
