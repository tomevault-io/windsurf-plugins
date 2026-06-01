---
trigger: always_on
description: Ikuyo is a collaborative trip/itinerary planning web application with real-time sync. Users create trips containing activities, accommodations, macroplans (high-level plans), expenses, tasks, and comments. It supports role-based access (Owner, Editor, Viewer) and sharing levels (Private, Group, Public).
---

# Ikuyo! (行くよ！) — Workspace Instructions for AI Coding Agents

Ikuyo is a collaborative trip/itinerary planning web application with real-time sync. Users create trips containing activities, accommodations, macroplans (high-level plans), expenses, tasks, and comments. It supports role-based access (Owner, Editor, Viewer) and sharing levels (Private, Group, Public).

## Tech Stack

| Layer              | Technology                                                 |
| ------------------ | ---------------------------------------------------------- |
| Language           | TypeScript (strict mode, ES2022)                           |
| Framework          | React 19                                                   |
| Bundler            | Rsbuild (Rspack-based — **not** Vite)                      |
| UI Library         | Radix UI Themes + Radix UI Primitives                      |
| Styling            | CSS Modules (`.module.css` / `.module.scss`), Radix CSS vars |
| State Management   | Zustand 5 (slice pattern with `persist` middleware)        |
| Database / Backend | InstantDB (`@instantdb/core` — real-time client-side DB)   |
| Routing            | Wouter (lightweight React router)                          |
| Date/Time          | Luxon (`DateTime`)                                         |
| Maps               | MapTiler SDK + Geocoding Control                           |
| Drag & Drop        | `@dnd-kit/core` + `@dnd-kit/sortable`                     |
| Error Monitoring   | Sentry (`@sentry/react`)                                   |
| Linting/Formatting | Biome (replaces ESLint + Prettier)                         |
| Testing            | Vitest + Testing Library (React) + jsdom                   |
| Git Hooks          | Lefthook (pre-commit: `biome check --write`)               |
| Package Manager    | pnpm 10                                                    |

## Project Structure

Feature-based folder structure under `src/`. Each domain feature is a **PascalCase** folder:

```
src/
  Activity/           # Feature folder
    Activity.tsx              # Main component
    Activity.module.css       # CSS Module
    db.ts                     # Database operations (InstantDB queries/mutations)
    time.ts                   # Time utilities for this feature
    ActivityNewDialog.tsx     # "New" dialog (imperative, pushed to dialog stack)
    ActivityDialog/           # View/Edit/Delete dialog subfolder
    ActivityForm/             # Form subfolder (with co-located tests)
  Accommodation/      # Same pattern as Activity
  Trip/               # Trip feature with store/ subfolder for Zustand slices
  Comment/
  Expense/
  Macroplan/
  Task/
  Auth/               # Authentication components and hooks
  common/             # Shared utilities, hooks, reusable UI components
  data/               # DB init, central Zustand store, shared types
  Dialog/             # Dialog system (route-based and imperative)
  Routes/             # Route definitions and constants
  theme/              # Theme system (light/dark)
  Toast/              # Toast notification system
```

Key conventions:
- **Features** are top-level PascalCase folders
- **Shared utilities** live in `src/common/`
- **Data layer** in `src/data/` (DB init, central store, shared types)
- **Tests** are **co-located** with source files: `*.test.ts` / `*.test.tsx`
- Each feature has its own `db.ts` for database operations

## Component Conventions

### Naming
- Components use named functions: `function ActivityInner({...}) {}`
- Inner components use `*Inner` suffix, wrapped with `memo()`: `export const Activity = memo(ActivityInner)`
- Page-level components use `Page*` prefix: `PageTrips`, `PageTrip`, `PageAccount`
- **Default exports** only for lazy-loaded page components (`React.lazy()`). All other components use **named exports**.

### CSS Modules
- Import alias is **`s`**: `import s from './Component.module.css'`
- Class names are camelCase: `s.activity`, `s.accommodationNotes`
- CSS uses Radix CSS custom properties: `var(--gray-7)`, `var(--accent-9)`, `var(--color-panel-solid)`
- Supports both `.module.css` and `.module.scss`

### Component Structure
1. External library imports (Radix, Luxon, React, etc.)
2. Internal imports (relative, from other features)
3. CSS module import
4. Types/interfaces (inline)
5. Inner component function
6. Memoized export

### Radix UI Usage
- Layout: `Box`, `Flex` from `@radix-ui/themes`
- Typography: `Text`, `Heading` from `@radix-ui/themes`
- Interactive: `Button`, `Dialog`, `ContextMenu`, `Switch`, `TextField`, `TextArea`
- Icons: `@radix-ui/react-icons`
- Utility: `clsx` for conditional class joining

## Database (InstantDB)

This project uses **InstantDB** (`@instantdb/core`) — a real-time, client-side database with no REST/GraphQL API layer. All data access happens via InstantDB's real-time sync.

For latest InstantDB docs, see https://www.instantdb.com/llms-full.txt 

### Singleton DB Instance

Defined in `src/data/db.ts`:

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenrick95/ikuyo](https://github.com/kenrick95/ikuyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
