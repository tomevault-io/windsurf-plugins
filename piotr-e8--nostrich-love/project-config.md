---
trigger: always_on
description: > **Nostrich.love** is a beginner-friendly educational platform for Nostr (Notes and Other Stuff Transmitted by Relays), a decentralized social media protocol. Uses Astro + React + Tailwind with i18n support for 7 languages (including RTL Arabic and Hindi).
---

# Nostrich.love - Agent Rules

> **Nostrich.love** is a beginner-friendly educational platform for Nostr (Notes and Other Stuff Transmitted by Relays), a decentralized social media protocol. Uses Astro + React + Tailwind with i18n support for 7 languages (including RTL Arabic and Hindi).

---

## Philosophy, Mission & Essence

**Core Philosophy:**  
Nostr shouldn't require technical expertise to use. Decentralization is meaningless if only developers can access it.

**Mission:**  
Make Nostr onboarding so smooth that creators (writers, artists, musicians) can start using it without friction, confusion, or fear.

**Essence:**  
An interactive, hands-on learning platform where you **try Nostr before committing to it** - test clients in browser, find curated communities, learn by experimentation.

**What it's all about:**
- **Human over protocol:** People first, technical specs second
- **Practice over theory:** Browser simulators, not documentation
- **Curation over chaos:** 300+ accounts organized by interest, not "figure it out yourself"
- **Accessibility over complexity:** 7 languages (including RTL Arabic and Hindi), visual guides, no assumptions

**The Stance:**  
Nostr keeps building infrastructure. Someone needed to build the **front door**.

**Why this matters for content:**  
Every post, guide, and feature should embody this philosophy. We're not explaining Nostr - we're making it usable.

---

## Critical Rules - READ FIRST

### 1. Internationalization is MANDATORY

**⚠️ NEVER hardcode strings in components.** Always use the translation system.

**7 Locales:** `en` (English), `pl` (Polish), `es` (Spanish), `de` (German), `zh` (Chinese), `ar` (Arabic - RTL), `hi` (Hindi)

```typescript
// Client-side (React)
const { t } = useTranslation();
<button>{t('ui.buttons.submit')}</button>

// Server-side (Astro)
const translations = getTranslations(currentLocale);
const title = translations.guides.whatIsNostr?.title;
```

**Translation Files:** `/src/i18n/locales/{en,pl,es,de,zh,ar,hi}.json`

---

### ⚠️ MANDATORY: New Locale Checklist (6 Hardcoded Files)

**When adding a new locale, these 6 files ALL have hardcoded locale arrays that MUST be updated — missing any one causes 404s or missing language switcher entries:**

| # | File | What to Update |
|---|------|---------------|
| 1 | `src/components/LanguageSwitcher.tsx` | Add to `languages` array, URL detection regexes, redirect patterns, localStorage check |
| 2 | `src/i18n/index.ts` | Add import, translations record entry, `getCurrentLocale()` detection |
| 3 | `src/pages/[lang]/guides/[slug].astro` | Add locale to `getStaticPaths()` locales array |
| 4 | `src/pages/[lang]/guides/index.astro` | Add params entry + locale detection if/else |
| 5 | `src/pages/guides/index.astro` | Add to localStorage saved language check array |
| 6 | `src/pages/progress.astro` | Add to saved language preference check array |

Plus the standard config files:
- `src/config/locales.ts` — Add locale entry with direction
- `src/i18n/types.ts` — Add locale string to Locale type union
- `astro.config.mjs` — Add locale string to `i18n.locales` and sitemap `i18n.locales`
- `scripts/verify-seo.js` — Add locale to check arrays
- `src/i18n/locales/{locale}.json` — Complete translation file
- `src/content/guides/{locale}/` — All 16 guide MDX files

**This is the #1 source of bugs when adding locales. Every previous locale addition (pl, es, de, zh, ar, hi) forgot at least one of these files.**

### 2. Guide Links MUST Include Locale Prefix

**❌ WRONG:** `/guides/what-is-nostr`
**✅ CORRECT:** `/en/guides/what-is-nostr` or `/${locale}/guides/what-is-nostr`

### 3. Dark Mode Colors

**❌ AVOID:** `dark:bg-gray-900/50` (creates muddy brown)
**✅ USE:** `dark:bg-gray-900` (solid dark gray)

### 4. Build Verification is REQUIRED

```bash
npm run build
```

Watch for: "Translation key not found" warnings, TypeScript errors, link validation issues.

### 5. File Scope Limits

- **Maximum 3 files per task**
- **1 file at a time** when creating new content
- Break complex tasks into sequential small tasks
- Build after EVERY component creation or guide file

### 6. Placeholder Syntax Patterns

**CRITICAL: The codebase uses TWO different placeholder conventions**

**Quiz components (double braces):**
```typescript
// Uses {{double}} braces
t('quiz.progress', { current: '{{current}}', total: '{{total}}' })
// Code does: text.replace("{{current}}", value)
```

**Navigation/other components (single braces):**
```typescript
// Uses {single} braces
t('navigation.level', { level: '{level}' })
// Code does: text.replace('{level}', value)
```

**Rule:** When creating or fixing translations with placeholders, check the component code to determine which syntax to use.

### 7. Verify Translation Structure Against Component Code

**Task agents can create wrong translation structures.**

**Example:** Agent created `nip05Checker.messages.*`, `nip05Checker.instructions.*`

**But component expected:** `nip05Checker.benefits.*`, `nip05Checker.form.*`, `nip05Checker.results.*`

**Always verify:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piotr-e8/nostrich-love](https://github.com/piotr-e8/nostrich-love) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
