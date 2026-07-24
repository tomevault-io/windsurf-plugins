---
trigger: always_on
description: A complete internal guide for contributors to the `supermarsx/sora-json-prompt-crafter` project. Built as a web-based interface with a focus on **privacy-first** prompt configuration for Sora’s generative AI models, it follows a component-driven modular structure (in spirit of the AgentsMD style). This guide covers project layout, component patterns, state management, development workflow, and quality standards.
---

# Sora JSON Prompt Crafter Development Guide

A complete internal guide for contributors to the `supermarsx/sora-json-prompt-crafter` project. Built as a web-based interface with a focus on **privacy-first** prompt configuration for Sora’s generative AI models, it follows a component-driven modular structure (in spirit of the AgentsMD style). This guide covers project layout, component patterns, state management, development workflow, and quality standards.

---

## 🔧 Tech Stack

- **Platform:** Browser (Progressive Web App; offline-capable via Service Worker)
- **Language:** TypeScript (strict mode)
- **Framework:** React 18 (functional components + hooks)
- **UI Library:** Shadcn UI (Radix Primitives) + Tailwind CSS
- **Build Tool:** Vite 5 (Module bundler and dev server)
- **Testing:** Jest + React Testing Library (unit and component tests)
- **Linting/Formatting:** ESLint and Prettier (enforced via scripts)

_(No dedicated backend – all logic runs client-side. External calls are limited to optional analytics or fetching repo stats.)_

---

## 📁 Project Structure

```
sora-json-prompt-crafter/
├── public/                 # Static assets (manifest, service worker, icons, disclaimer)
├── src/
│   ├── components/         # UI components (app screens, modals, panels, controls)
│   │   └── ui/             # Reusable base components (buttons, cards, dialogs, etc.)
│   ├── hooks/              # Custom React hooks (responsive layout, dark mode, tracking)
│   ├── lib/                # Core logic modules (JSON generation, default options, storage, analytics)
│   ├── data/               # Preset data and option definitions (e.g. camera angles, style presets)
│   └── index.tsx           # Application entry point (renders the main app component)
├── tests/                  # Unit/Integration tests (if not collocated with src/)
├── index.html              # HTML template for Vite (app mounting point and script includes)
├── package.json            # Project metadata and NPM scripts (dev, build, test, lint, format)
├── tsconfig.json           # TypeScript configuration (paths, strict settings)
├── tailwind.config.js      # Tailwind CSS configuration (theme and plugin settings)
└── vite.config.ts          # Vite configuration (build setup, aliases)
```

- **public/**: Contains static files served directly. Notably includes `sw.js` (the service worker enabling offline functionality), `site.webmanifest` (PWA manifest for installable app), icons, and the `disclaimer.txt` displayed in-app. A build step (`npm run generate-sw-assets`) scans `public/` for localized disclaimers and translations and outputs `public/sw-assets.js` consumed by `sw.js`.
- **src/**: All source code.
  - `components/`: React components for the UI. This includes higher-level components (e.g. **Dashboard** page, **HistoryPanel**, **ControlPanel**, **ShareModal**, etc.) as well as a `components/ui/` subfolder with low-level UI elements (buttons, cards, sliders, dialogs) following the design system.
  - `hooks/`: Custom hooks encapsulating cross-cutting logic (e.g. `useDarkMode` toggles the theme, `useTracking` manages analytics opt-in, `useIsSingleColumn` handles responsive layout changes, etc.).
  - `lib/`: Core logic and utilities. For example, **generateJson** (builds the JSON output from the options state), **defaultOptions** (default values for all Sora options), **validateOptions** (ensures loaded JSON is valid), **storage** (wrapper around localStorage for persistence), and **analytics** (for event tracking). These modules keep non-UI logic self-contained.
  - `data/`: Definition of option values and presets. This includes structured lists for various selectable parameters (camera types, lens options, quality levels, fantasy presets, etc.). By organizing them here, the app can easily extend or modify preset categories without cluttering core logic.
  - `index.tsx`: The app’s entry file. It initializes the React app, rendering the main component (often the Dashboard or App) into the `index.html` container. Also applies global providers (if any) and imports global styles.

- **tests/**: Contains test files (if separated; e.g. `*.test.ts` or `*.spec.tsx`). The project uses **Jest** and React Testing Library to verify component rendering and **generateJson** logic. (In smaller projects, tests may reside alongside their implementation in `src/`.)
- **index.html**: The root HTML page used by Vite. It loads the React bundle and includes references to the manifest, fonts, and sets up the basic HTML skeleton.
- **package.json**: Defines dependencies and NPM scripts. Key scripts include `npm run dev` (launches Vite dev server), `build` (produces a production build in the `dist/` directory), `test` (runs the Jest test suite), `lint` (ESLint), `format` (Prettier), and `typecheck` (TypeScript compiler for type checking).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supermarsx/sora-json-prompt-crafter](https://github.com/supermarsx/sora-json-prompt-crafter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
