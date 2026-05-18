---
trigger: always_on
description: These rules define the development patterns, conventions, and architecture for Sprout Track. This document serves as the primary context for AI-assisted development sessions.
---

# Next.js Development Rules

These rules define the development patterns, conventions, and architecture for Sprout Track. This document serves as the primary context for AI-assisted development sessions.

## Tech Stack

- Next.js with App Router: Core framework for routing, server components, and API routes.
- TypeScript: For type-safe code.
- Prisma ORM with support for both PostgreSQL and SQLite for data persistence. All queries and schema must be compatible with both database systems.
- TailwindCSS: For utility-first styling (no CSS Modules, no Styled Components).
- React Hooks (useState, useEffect, useContext): Used for all client-side state management, data fetching, and form handling.
- PWA architecture with offline support, push notifications (VAPID), and Wake Lock API.

## Project Structure

- Follow the `/src` directory structure with dedicated folders for components, hooks, services, utils, constants, context, types, and styles
- Component organization under `src/components/`:
  - `ui/` — base UI primitives (button, input, etc.), each with their own folder containing `index.tsx`, `styles.ts`, `types.ts`, `.css`, and `README.md`
  - `forms/` — form components built on top of `ui/` form page components
  - `modals/` — modal components
  - Feature components live in their own named folders at the component root (e.g., `Calendar/`, `Timeline/`, `Reports/`, `DailyStats/`, `BabySelector/`, `SetupWizard/`) — there is no `features/` directory
  - Domain management components: `account-manager/`, `familymanager/`
- Keep component files, styles, types, and documentation together in the same folder
- Include README.md files for components documenting props, usage, and implementation details
- Maintain a consistent naming convention across all files and components
- Backend API layers are fully separated from the Next.js web layer

## Component Architecture

- Create “dumb” UI components that are highly configurable through props
- Implement container/presentational pattern to separate data fetching from UI rendering
- Keep components small and focused (under 200 lines of code)
- Use composition over inheritance when building complex components
- Include props for error and loading states in all data-dependent components
- Design with accessibility in mind from the start (proper ARIA attributes)
- Document component APIs thoroughly in component README files

## TypeScript Implementation

- Enable strict TypeScript mode in `tsconfig.json`
- Create shared types in the `/types` directory for domain entities
- Use discriminated unions for different event types (e.g., feeding vs. diaper events)
- Implement type guards for safe type narrowing
- Define constants with type safety using `as const` assertions
- Create specific error types that extend the base Error class
- Use TypeScript generics for reusable components and functions
- Define readonly properties for immutable data

## State Management

- Implement custom hooks for complex state logic
- Create context providers for global state that needs to be accessed by multiple components
- Use `useEffect` for data fetching with proper loading and error state handling — React Query is not used in this project
- Follow immutable update patterns in all state modifications
- Implement memoization for expensive computations with useMemo
- Include proper error and loading states in all async operations
- Design state management with offline-first principles

## Styling Approach

- Do not use CSS Modules (`.module.css`), Styled Components, or inline style objects
- Components use Class Variance Authority (CVA) for variant management
- Each component follows a modular file structure:
  - `index.tsx` — component implementation
  - `[component].styles.ts` — CVA style definitions with Tailwind classes for light mode variants
  - `[component].css` — plain CSS file with `html.dark` selectors for dark mode overrides using custom class names (e.g., `.button-dark-outline`, `.button-dark-ghost`)
  - `[component].types.ts` — TypeScript type definitions
- **Dark mode uses `html.dark` CSS selectors in plain `.css` files, not Tailwind `dark:` classes.** This is intentional — Tailwind’s `dark:` classes respond to system preferences, which bypasses the in-app theme toggle. The `html.dark` class is controlled by the app’s theme context so the toggle works correctly regardless of system settings. Do not use `dark:` prefixed Tailwind classes.
- Light mode = Tailwind utilities via CVA in `styles.ts`. Dark mode = `html.dark .class-name` overrides in `[component].css`. Both are applied to the component together.
- Define a shared design system with consistent variables for colors, spacing, and typography
- Keep styles modular and component-specific using Tailwind utility classes
- Implement responsive design using Tailwind breakpoints
- Export theme tokens as TypeScript constants
- Create helper functions for complex style logic when Tailwind alone is insufficient

## Navigation

- Use Next.js’s file-system based routing
- Define strongly typed route parameters with TypeScript interfaces
- Create navigation utilities that consolidate route definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oak-and-Sprout/sprout-track](https://github.com/Oak-and-Sprout/sprout-track) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
