---
trigger: always_on
description: React + TypeScript + Vite PWA that displays a personal CV/resume.
---

# see-veo

React + TypeScript + Vite PWA that displays a personal CV/resume.

## Tech Stack

- React 19 with TypeScript
- Vite 7 with `@vitejs/plugin-react`
- Tailwind CSS v4 via `@tailwindcss/vite` (no tailwind.config — all config is CSS-first in `src/index.css` using `@theme`)
- PWA via `vite-plugin-pwa` (Workbox service worker, web manifest)
- Vercel deployment (auto-deploys on push to `main`, configured via `vercel.json`)

## Project Structure

- `src/data/cv-data.ts` — All CV content and TypeScript interfaces. Edit this single file to update the resume.
- `src/components/` — One component per CV section (Hero, About, Experience, Education, Skills, Projects) plus feature components (ActivityCharts, ActivityTimeline, InterestForm, DebugBanner, UpdatePrompt, InstallInstructionsModal, ProjectImage) and reusable helpers (Section, TimelineItem, SkillBadge).
- `src/hooks/` — Custom React hooks. `useRepoTorEmbed` loads repo-tor's `embed.js` helper script to auto-size chart iframes; `usePWAInstall` and `usePWAUpdate` for install/update prompts.
- `src/constants/embed.ts` — Centralized repo-tor embed configuration (base URL, script URL, chart colors).
- `src/utils/` — Shared utilities: `debugLog.ts` (pub/sub event store for mobile debugging), `pwa.ts` (browser detection, standalone check), `validation.ts` (email pattern, form payload validation), `fetchWithTimeout.ts` (fetch wrapper with automatic abort-on-timeout), `diagnostics.ts` (diagnostic check functions and failure diagnosis shared by DebugBanner and InterestForm).
- `src/index.css` — Tailwind import, custom `@theme` color tokens (dark palette), and print styles. Single source of truth for theme colors — `vite.config.ts` parses this file to feed PWA manifest and HTML meta tags.
- `src/App.tsx` — Composes all sections into a single-page layout. No routing.
- `vite.config.ts` — Vite config with Tailwind plugin, PWA plugin, `themeColorInjector` plugin (injects theme colors into HTML), and Workbox runtime caching rules.
- `vercel.json` — Vercel deployment config with SPA rewrites.
- `vitest.config.ts` — Vitest config with jsdom environment, React plugin, and setup file (`src/test/setup.ts`).
- `src/test/` — Test files (Vitest + Testing Library).
- `.env.example` — Example environment variables (`VITE_INTEREST_API_URL`).

## Commands

- `npm run dev` — Start dev server
- `npm run type-check` — TypeScript type check only (no build)
- `npm run build` — TypeScript check + production build (`tsc -b && vite build`)
- `npm run lint` — Run ESLint
- `npm run preview` — Preview production build locally
- `npm run test` — Run Vitest test suite
- `npm run test:watch` — Run Vitest in watch mode

## Key Decisions

- Dark minimal theme (no light/dark toggle). Near-monochrome neutral grays defined via Tailwind v4 `@theme` tokens. Project cards use per-project accent colors (stored in `cv-data.ts`) for visual identity.
- Single-page app with no client-side routing.
- PWA `scope` and `start_url` use `/` — Vercel serves at root, no base-path prefix needed.
- Print styles in `src/index.css` override to white background. Elements with class `no-print` are hidden when printing.
- Interest notification form (`InterestForm` component, rendered inline after Education) — POSTs to an **external** API endpoint (separate project, not part of this repo) configured via `VITE_INTEREST_API_URL` env variable. Hidden when printing. Degrades gracefully when API is not configured or offline. Client-side validation via shared `validatePayload` utility before network requests.

---

## Hard Rules

These rules are non-negotiable. Stop and ask before proceeding if any rule would be violated.

### Before Making Changes

- Read relevant existing code and documentation first
- Ask clarifying questions if scope, approach, or intent is unclear
- Confirm understanding before implementing non-trivial changes
- Never assume — when in doubt, ask

### Best Practices

- Follow established patterns and conventions already in the codebase
- Use industry-standard solutions over custom implementations when available
- Prefer well-maintained, widely-adopted libraries over obscure alternatives
- Apply SOLID principles, DRY, and separation of concerns
- Follow security best practices (input validation, sanitization, principle of least privilege)
- Handle errors gracefully with meaningful messages
- Write self-documenting code with clear naming

### Code Organization

- Prefer smaller, focused files and functions
- Pause and consider extraction at: 500 lines (file), 100 lines (function), 400 lines (class)
- Strongly consider refactoring at: 800+ lines (file), 150+ lines (function), 600+ lines (class)
- Extract reusable logic into separate modules/files immediately
- Group related functionality into logical directories
- Split large classes into smaller, focused classes when responsibilities diverge

### Styling

- Use Tailwind CSS utility classes in JSX — this is the project's standard approach
- Custom theme tokens, base styles, and print overrides go in `src/index.css` via `@theme`
- Do not create separate component stylesheet files
- Do not write inline `style={}` attributes; use Tailwind classes instead

### Decision Documentation

Every non-trivial code change must include comments explaining:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devmade-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
