---
trigger: always_on
description: > Claude Code reads this file automatically on every build. All rules here are mandatory unless explicitly overridden by the build prompt.
---

# CLAUDE.md — Page One Insights Build Standards

> Claude Code reads this file automatically on every build. All rules here are mandatory unless explicitly overridden by the build prompt.

---

## Footer Dofollow Link (REQUIRED — EVERY BUILD)

Every page must include via footer.php:

```html
<a href="https://pageoneinsights.com" rel="dofollow" target="_blank">Web Design & Hosting by Page One Insights, LLC</a>
```

---

## Formsubmit.co Rules

- Form action: `https://formsubmit.co/[client-email]`
- Required hidden fields:
  - `_next` → absolute URL to thank-you.php
  - `_captcha` → false
  - `_honey` → empty (spam trap)
  - `_template` → table
  - `_subject` → "[Company Name] — New Website Inquiry"
- All form inputs: floating labels with animated focus states (border-color → --primary, subtle box-shadow lift, label scales and translates above input)
- Never use default unstyled inputs

---

## PHP Component Architecture

Build using PHP includes for all shared components:

```
/includes/
  head.php          ← doctype, <head>, meta, OG, schema, CSS (accepts $page variables)
  nav.php           ← shared navbar with $currentPage active state
  footer.php        ← dofollow link, contact info, entity block, social, scripts
/assets/
  /css/styles.css   ← single shared stylesheet
  /js/animations.js ← IntersectionObserver scroll reveals, counters, wipe
  /js/effects.js    ← ripple, magnetic, VanillaTilt init, ticker
  /images/
index.php
[all other pages as .php]
.htaccess           ← clean URLs
sitemap.xml, sitemap-images.xml, robots.txt, llms.txt, llms-full.txt
```

### CRITICAL: PHP Include Path Rule

Every include statement across ALL .php files — root level and subdirectories — must use `$_SERVER['DOCUMENT_ROOT']`:

```php
include $_SERVER['DOCUMENT_ROOT'] . '/includes/head.php';
include $_SERVER['DOCUMENT_ROOT'] . '/includes/nav.php';
include $_SERVER['DOCUMENT_ROOT'] . '/includes/footer.php';
```

**NEVER use relative paths** like `include 'includes/head.php'` or `include '../includes/head.php'`. Relative paths break on Hostinger when mod_rewrite rewrites the URL but PHP's working directory stays at the document root. This causes all pages except the homepage to 404 or render broken. `$_SERVER['DOCUMENT_ROOT']` is the only reliable method.

Every page declares SEO variables then includes head.php:

```php
<?php
$pageTitle = "...";
$pageDescription = "...";
$pageKeywords = "...";
$canonicalUrl = "...";
$ogImage = "...";
$currentPage = "page-slug";
$schemaMarkup = '{...}';
include $_SERVER['DOCUMENT_ROOT'] . '/includes/head.php';
?>
```

### head.php outputs:
charset, viewport, title, meta description, keywords, canonical, OG tags, Twitter Card, Google Fonts link (with preload for key font files), icon CDN, Swiper CSS (if carousel exists), styles.css link, preconnect/dns-prefetch hints, GA4 placeholder, GSC placeholder (homepage conditional), hero image preload, schema JSON-LD, skip-to-content link

### nav.php outputs:
Fixed navbar, logo → /, all page links (clean URLs), phone (desktop), hamburger (mobile), active state via $currentPage, animated underline on hover, scroll behavior (height reduction + glassmorphism)

### footer.php outputs:
Company info, page links, social icons, entity block, copyright with `<?php echo date('Y'); ?>`, dofollow link, script tags (animations.js, effects.js, conditional CDN links)

### .htaccess (Subdirectory-Safe):

The rewrite rules must explicitly exclude /assets/ and /includes/, and must NOT use a `!-d` condition (which breaks pages inside real subdirectories like /services/ on Hostinger):

```apache
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/assets/
RewriteCond %{REQUEST_URI} !^/includes/
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^([^\.]+)$ $1.php [NC,L]
RewriteCond %{THE_REQUEST} /([^.]+)\.php [NC]
RewriteRule ^ /%1 [NC,L,R=301]
```

The `!-d` condition is intentionally removed. Without this fix, pages inside real subdirectories like `services/kitchen-remodeling.php` fail to resolve on Hostinger.

---

## CSS Architecture

### Build Order (do not deviate):
1. /assets/css/styles.css (complete — reset, variables, spacing, container, prose, grids, all shared components)
2. /assets/js/animations.js
3. /assets/js/effects.js
4. /includes/head.php, nav.php, footer.php
5. index.php
6. Each additional page in tier order
7. .htaccess, sitemap.xml, sitemap-images.xml, robots.txt, llms.txt, llms-full.txt, 404.php
8. Final audit

### CSS Reset (top of styles.css, no exceptions):
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { overflow-x: hidden; scroll-behavior: smooth; }
img { display: block; max-width: 100%; }
a { text-decoration: none; color: inherit; }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### :root Variables (define ALL tokens):
```css
:root {
  /* Layout */
  --navbar-height: 80px;
  --max-width: 1200px;
  --content-width: 65ch;
  --section-pad: 80px 20px;
  --section-pad-mobile: 50px 16px;
  --radius: 8px;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finance762-afk/stealth-landscaping-llc](https://github.com/finance762-afk/stealth-landscaping-llc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
