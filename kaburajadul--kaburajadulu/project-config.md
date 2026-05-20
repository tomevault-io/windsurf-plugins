---
trigger: always_on
description: Agent instructions for the KaburAjaDulu codebase — an Astro 5 platform helping Indonesians
---

# AGENTS.md

Agent instructions for the KaburAjaDulu codebase — an Astro 5 platform helping Indonesians
explore study and work opportunities abroad. Domain: **kaburajadulu.com**

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Astro 5 (static output, no SSR adapter) |
| UI | React 19 + ShadCN (New York style) + Radix UI |
| Styling | Tailwind CSS v4 via `@tailwindcss/vite` (no `tailwind.config.js`) |
| Language | TypeScript (strict mode via `astro/tsconfigs/strict`) |
| Package manager | Bun |
| Deployment | Cloudflare Workers (static assets) |

---

## Development Commands

```bash
# Install dependencies
bun install

# Start development server (http://localhost:4321)
bun dev

# Type-check the entire project
bun run astro check

# Build for production (output to ./dist)
bun run build

# Preview production build locally (standard Node preview)
bun run preview

# Raw Astro CLI passthrough
bun run astro
```

---

## Cloudflare Deployment

The site deploys as a **static site** to Cloudflare Workers. No SSR adapter is needed.

```bash
# Install Wrangler CLI (already in devDependencies)
bun install

# Preview production build locally using Cloudflare's workerd runtime
bun run cf:preview

# Build and deploy to Cloudflare Workers (kaburajadulu.com)
bun run deploy
```

### First-time setup

1. Authenticate with Cloudflare:
   ```bash
   bunx wrangler login
   ```
2. Run the initial deploy:
   ```bash
   bun run deploy
   ```
3. In the Cloudflare dashboard → Workers & Pages → `kaburajadulu` → Custom Domains,
   add `kaburajadulu.com` and `www.kaburajadulu.com`.

### Configuration files

- `wrangler.jsonc` — Cloudflare Workers config (name, compatibility date, assets dir)
- `public/_headers` — custom HTTP response headers for static assets
- `public/_redirects` — custom redirects for static assets

### Environment secrets

For any future server-side secrets, use Wrangler (never commit to repo):
```bash
bunx wrangler secret put SECRET_NAME
```

For local dev secrets, create `.dev.vars` (already gitignored):
```
MY_SECRET=value
```

---

## Internationalization (i18n)

The site supports 13 languages via URL-prefix routing. All UI strings are translated via `react-i18next`.

### Supported Locales

| Code | Language | Direction |
|---|---|---|
| `id` | Indonesian (default) | LTR |
| `en` | English | LTR |
| `ja` | Japanese | LTR |
| `zh-cn` | Chinese (Simplified) | LTR |
| `zh-tw` | Chinese (Traditional) | LTR |
| `ko` | Korean | LTR |
| `es` | Spanish | LTR |
| `ar` | Arabic | **RTL** |
| `nl` | Dutch | LTR |
| `it` | Italian | LTR |
| `de` | German | LTR |
| `fr` | French | LTR |
| `sv` | Swedish | LTR |

### URL Structure

```
/                   → Indonesian (default, no prefix)
/en/               → English
/ja/               → Japanese
/zh-cn/            → Chinese Simplified
/zh-tw/            → Chinese Traditional
...etc.
/blog/post-slug    → Indonesian blog
/en/blog/post-slug → English blog
```

### Adding a New Locale

1. Create `src/locales/{code}.json` with all UI strings
2. Add the locale code to `SUPPORTED_LANGUAGES` in `src/i18n/config.ts`
3. Add locale to `LOCALES` in `src/i18n/constants.ts`
4. Add country→locale mapping in `src/i18n/localeMapping.ts`
5. Add flag and name in `src/i18n/constants.ts`
6. If RTL, add to `LOCALE_DIR` mapping with `rtl`
7. Add hreflang in `Layout.astro` if not already covered
8. Add new page routes in `src/pages/[lang]/` if needed

### Geo-Detection

The `Layout.astro` includes an inline geo-detection script that reads `CF-IPCountry` (set by Cloudflare) or falls back to `navigator.language`. The detected locale is set as `data-detected-locale` on `<html>`. The `LanguageSwitcher` uses this to highlight the detected language in the dropdown.

### RTL Support

Arabic (`ar`) sets `dir="rtl"` on `<html>`. RTL CSS overrides are in `Layout.astro`'s `<style is:global>`. Key patterns: `.navbar { flex-direction: row-reverse; }`, `.ml-auto` → `.mr-auto`, etc.

### Testing i18n Locally

```bash
bun dev
# Visit: http://localhost:4321/ja/
# Visit: http://localhost:4321/ar/
# Visit: http://localhost:4321/zh-tw/
```



No test framework is currently configured. When adding tests:

```bash
# Recommended: Vitest (compatible with Vite/Astro)
bun add -D vitest @testing-library/react

# Run all tests
bunx vitest run

# Run a single test file
bunx vitest run src/components/blog/BlogCard.test.tsx

# Run tests in watch mode
bunx vitest
```

Type-check (substitute for lint until ESLint is configured):
```bash
bun run astro check
```

---

## Project Structure

```
src/
├── components/
│   ├── blog/          # BlogCard.tsx, BlogSection.tsx
│   ├── home/          # HeroSection, AboutSection, DestinationShowcase, CTASection
│   ├── layout/        # Navbar.tsx, Footer.tsx, LanguageSwitcher.tsx
│   └── ui/            # ShadCN primitives: button, card, badge, aspect-ratio
├── constants/
│   └── urls.ts        # All external/internal URLs — add new URLs here
├── content/
│   ├── blog/          # Markdown blog posts (.md)
│   └── config.ts      # Zod schema for content collections
├── i18n/
│   ├── index.ts           # Re-exports
│   ├── config.ts          # react-i18next setup
│   ├── constants.ts       # LOCALE_NAMES, LOCALE_FLAGS, LOCALE_DIR, LOCALES

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaburAjaDul/kaburajadulu](https://github.com/KaburAjaDul/kaburajadulu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
