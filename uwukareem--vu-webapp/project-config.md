---
trigger: always_on
description: **VU** is an AI-powered virtual interview platform with two modes: **Practice Mode** (job seekers, mock interviews) and **Recruitment Mode** (companies, AI-led candidate screening). This React 19 SPA uses Vite 7, Tailwind CSS 4, and follows a design-first approach from [Figma](https://www.figma.com/design/LgLS6zCwbhl4yISLlsN2qC/VU-WebApp).
---

# Copilot Instructions for VU Frontend

## Project Overview

**VU** is an AI-powered virtual interview platform with two modes: **Practice Mode** (job seekers, mock interviews) and **Recruitment Mode** (companies, AI-led candidate screening). This React 19 SPA uses Vite 7, Tailwind CSS 4, and follows a design-first approach from [Figma](https://www.figma.com/design/LgLS6zCwbhl4yISLlsN2qC/VU-WebApp).

## Tech Stack & Tooling

| Technology       | Version | Purpose                                                              |
| ---------------- | ------- | -------------------------------------------------------------------- |
| **React**        | 19      | UI Framework                                                         |
| **Vite**         | 7       | Build tool & HMR dev server                                          |
| **Tailwind CSS** | 4       | Utility-first styling (via `@import 'tailwindcss'` + `@theme` block) |
| **Lucide React** | 0.562+  | Icon components (component-based, not font icons)                    |
| **PropTypes**    | 15.8+   | Runtime prop validation (required for all components)                |
| **ESLint**       | 9       | Linting (flat config in `eslint.config.js`)                          |
| **Prettier**     | 3.7     | Code formatting + Tailwind class sorting                             |

**No routing library yet** - manual state-based navigation in `App.jsx` (React Router planned).

## Development Commands

```bash
npm run dev       # Dev server on http://localhost:5173 (Vite HMR)
npm run build     # Production build to dist/
npm run preview   # Preview production build locally
npm run lint      # ESLint validation
```

**Note**: Prettier runs on save in VS Code. Tailwind classes are auto-sorted by `prettier-plugin-tailwindcss`.

---

## Architecture & File Organization

```
src/
├── components/
│   ├── ui/                    # Reusable UI primitives
│   │   ├── Badge/             # Badge + RoleBadge + variants.js
│   │   ├── Breadcrumb/
│   │   ├── Button/
│   │   ├── Cards/             # ActionCard, EntityCard, InfoCard, QuestionCard, QuickInfoCard
│   │   ├── Charts/            # DonutChart, StatsChart
│   │   ├── Input/             # Input, InputField, Label, Hint + variants.jsx
│   │   ├── Pagination/
│   │   ├── SidebarButton/
│   │   ├── Tables/            # TableHeader, TableRow, TableCell
│   │   ├── Tabs/
│   │   ├── Tags/
│   │   ├── Toggle/
│   │   ├── User/
│   │   └── index.js           # Barrel export for all UI components
│   └── layout/                # Layout components
│       ├── Navbar/            # Navbar + NotificationDropdown
│       ├── PageLayout/        # Main app shell (sidebar + navbar + content)
│       ├── Shortcuts/         # Action bar (filters + search + buttons)
│       ├── Sidebar/
│       └── index.js           # Barrel export for layout components
├── pages/                     # Route-level page components
│   ├── Candidates/            # CandidatesPage + Pipeline/
│   ├── CompanyTeam/           # (placeholder)
│   ├── Jobs/                  # (placeholder)
│   ├── Mocks/                 # (placeholder)
│   ├── Profile/               # (placeholder)
│   └── _showcase/             # ComponentShowcase (demo page)
├── styles/
│   ├── index.css              # Tailwind CSS 4 import + @theme config + utilities
│   └── tokens.css             # 460+ Figma-exported CSS variables
└── assets/                    # Static images, icons
```

### Key Files

| File                                    | Purpose                                                        |
| --------------------------------------- | -------------------------------------------------------------- |
| `src/components/ui/index.js`            | Central barrel export - **add new UI components here**         |
| `src/components/layout/index.js`        | Barrel export for layout components                            |
| `src/styles/tokens.css`                 | 460+ design tokens from Figma - **single source of truth**     |
| `src/styles/index.css`                  | Tailwind CSS 4 config via `@theme` block + custom utilities    |
| `src/App.jsx`                           | Main entry point with state-based navigation in `renderPage()` |
| `src/pages/_showcase/ComponentShowcase` | Component demo page for testing                                |

---

## Component Patterns

### 1. Standard Component Structure

Every UI component follows this folder structure:

```
src/components/ui/{ComponentName}/
├── {ComponentName}.jsx     # Component logic + PropTypes
├── {ComponentName}.css     # BEM-like CSS with design tokens
└── index.js                # Re-export: export { ComponentName } from './{ComponentName}'
```

**Complete Example** (`src/components/ui/Button/Button.jsx`):

```jsx
import './Button.css';
import PropTypes from 'prop-types';

const VARIANTS = ['primary', 'secondary', 'ghost'];

export function Button({
  children,
  variant = 'primary',
  disabled = false,
  iconLeft,
  iconRight,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UwUkareem) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
