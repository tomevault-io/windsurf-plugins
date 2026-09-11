---
trigger: always_on
description: | Command | What it does |
---

# AGENTS.md — DevConnect AI

## Quick Commands

| Command | What it does |
|---|---|
| `npm run dev` | Dev server on `localhost:3000` (uses `--webpack` flag) |
| `npm run build` | Production build |
| `npm run lint` | Run ESLint |
| `npm run check:structure` | Validates required folders: `app`, `components`, `context`, `lib` |

**CI order:** `npm ci` → `check:structure` → `build`. Requires Firebase env vars as GitHub secrets (see `main.yml`).

## Architecture

**Stack:** Next.js 16 App Router, React 19, Tailwind CSS v4, Firebase (Auth + Firestore), Sarvam AI.

**Entry points:**
- `app/layout.js` — Root layout. Wraps app in `ThemeProvider` → `AuthProvider`. Injects theme `<script>` to prevent flash.
- `app/page.js` — Landing page (public). Large client component (~1000 lines of inline styles).
- `app/dashboard/page.js` — Main community feed (protected). ~1685 lines. This is the core of the app.
- `app/api/ai-draft/route.js` — Server-side AI endpoint. Calls Sarvam AI. API key is server-only (`SARVAM_API_KEY`, no `NEXT_PUBLIC_` prefix).
- `app/api/code-review/route.js` — Local regex-based code review (not AI-powered). Rate limited to 10/hour.

**Protected routes:** `/dashboard`, `/profile`, `/settings`, `/user/[uid]`. Guarded by `components/ProtectedRoute.js` which redirects to `/login`.

## Styling

Most UI uses **inline style objects** with CSS custom properties, not Tailwind utility classes. Tailwind v4 is imported in `globals.css` but primarily for reset/utility support. The design system is defined via CSS variables in `globals.css` (`:root[data-theme="dark"]` and `:root[data-theme="light"]`).

When adding or modifying UI, reference the existing `S` style objects in each component. Use CSS variables (`var(--bg-primary)`, `var(--accent-primary)`, etc.) for theme-aware values.

## Theme System

- Defaults to **dark mode**.
- Persisted in `localStorage` under key `"theme"`.
- Applied via `data-theme` attribute on `<html>`.
- `ThemeContext` provides `{ isDarkMode, toggleTheme }`.
- Inline theme colors also mirrored in `lib/theme.js` (for programmatic access).

## Firebase

- Initialized client-side only (`lib/firebase.js` checks `typeof window`).
- Auth providers: Google, GitHub, email/password.
- User profiles saved to Firestore `users` collection on first login via `setDoc` with `merge: true`.
- Real-time updates use Firestore `onSnapshot` (dashboard feed).
- All Firebase config comes from `NEXT_PUBLIC_FIREBASE_*` env vars.

## Key Conventions

- All client components must have `"use client"` directive.
- ESM modules (`"type": "module"` in `package.json`).
- PostCSS config: `postcss.config.mjs` with `@tailwindcss/postcss` plugin (Tailwind v4, no `tailwind.config.js`).
- No `next.config.js` or `next.config.mjs` exists.
- Branch naming: `feat/`, `fix/`, `docs/` prefixes. Commit convention: `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `chore:`.

## Installed Skills

| Skill | Use Case |
|---|---|
| `wcag-audit-patterns` | Audit pages for WCAG 2.2 accessibility violations, fix issues, meet ADA/Section 508 compliance |
| `core-web-vitals` | Diagnose and fix LCP, INP, CLS — improves page experience and Google Search ranking |
| `seo-audit` | Full technical SEO audit: meta tags, crawl errors, indexing, traffic drops, on-page issues |
| `conventional-commit` | Generate standardized commit messages following the Conventional Commits spec |

## Gotchas

- **No `.env.example` file** exists despite README referencing it. Create `.env.local` manually with the required vars.
- **`hooks/` directory** exists but is NOT in the `check:structure` required folders list.
- **AI Draft** uses `sarvam-30b` model. If you see "model deprecated" errors, update the model name in `app/api/ai-draft/route.js`.
- **Code review endpoint** is regex-based, not AI. It detects `eval()`, `innerHTML`, `var`, `console.log`, nested loops, and language detection.
- **Dark mode flash prevention:** The root layout injects an inline `<script>` that reads `localStorage` before React mounts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ARUNNG2004/DevConnect-AI](https://github.com/ARUNNG2004/DevConnect-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
