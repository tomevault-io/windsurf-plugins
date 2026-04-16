---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a monorepo for "Min or Max", a multiplayer card game where you lay low or play it to the max!

The web application serves as its own marketing site, and includes the game itself, utilising Cloudflare features for real-time play. Built with SvelteKit and modern web technologies.

Locally, the dev server is always running at http://localhost:5173/

## Coding Standards

- **Strongly typed**: Consistently use type aliases for all object type definitions
- You MUST type things accurately. Don't be lazy.
- You must NOT use the `any` type, or similarly lazy ways out.
- **You MUST design for a single source of truth**: DO NOT hardcode strings and arrays when data can be derived.
- **NEVER** try format code or check for whitespace yourself. Use your Write & Edit tools properly. We have a dedicated formatter you may use at the end of your tasks.
- It will **NEVER** be a formatting issue.
- You must **NEVER** over-indulge with comments! If you find yourself writing comments, stop and think whether a better variable or function name would make the code clearer instead.
- You must **NEVER** write comments which only explain the following line of code!

## Architecture

### Monorepo Structure

- `apps/web/` - Main SvelteKit application
- `apps/api/` - Cloudflare Worker using Durable Objects to communicate game state
- `packages/state/` - State shared by web & server apps
- `packages/rng/` - Utility for seeded randomness that can run on the server or browser

## Development Commands

All commands should be run from the repository root:

- `pnpm dev` - Start all development servers in watch mode
- `pnpm build` - Build all apps for production
- `pnpm test` - Run all tests (requires dev servers running)
- `pnpm lint` - Lint all code
- `pnpm format` - Format all code with Prettier

## Web App (apps/web/)

### Web App Specific Commands

From `apps/web/` directory:

- `pnpm test:unit` - Run Vitest unit tests
- `pnpm test` - Run all Playwright tests
- `pnpm check` - Type check with svelte-check

### Tech Stack

- **Framework**: SvelteKit with Svelte 5 (Runes)
- **Styling**: Tailwind CSS v4 + `shadcn-svelte` components
- **Testing**: Playwright (e2e/integration), Vitest (unit)
- **Icons**: Unplugin Icons
- **Deployment**: Cloudflare Workers

### Key Directories

- `src/lib/components/` - Reusable Svelte components
- `src/routes/` - SvelteKit file-based routing
- `tests/` - Playwright test suites

### Testing Strategy

- **Playwright**: e2e and integration tests, run with dev servers
- **Vitest**: Unit tests, can run independently

You MUST start Playwright specs / tests by navigating to a page, then:

```ts
await expect(page.getByTestId('hydrated')).toBeVisible()
```

This ensures our app is fully interactive.

Prefer tests that tell a bigger story: Have the "user" (as Playwright) interact with the application for a realistic flow.

#### Example

Test the happy path in a single test:

- "I expect to see this", then "I type this", then "I click this", then "I expect to see this"
- AVOID individual tests for a single action "I expect to see this"

#### Meaningful tests

Even the Vitest tests have to be meaningful!

**NEVER write tests that only verify types or function existence**. TypeScript already guarantees these at compile time. Tests MUST verify actual behaviour, logic, edge cases, or integration points.

Examples of USELESS tests to NEVER write:

- `expect(typeof someFunction).toBe('function')` - TypeScript already checks this
- `expect(someFunction).toBeDefined()` - TypeScript already checks this
- `expect(someObject).toHaveProperty('field')` - TypeScript already checks this

If you have to write a unit test, it must answer: "What bug would this catch that TypeScript can't?"

EVERY test should have a realistic to see fail.

### Environment Setup

- Use pnpm as package manager
- Node.js 25 required (managed by Mise)

### Tailwind CSS v4 Configuration

- **No tailwind.config file**: Tailwind v4 does not use a traditional config file
- **CSS-based configuration**: All Tailwind configuration is done in CSS files
- **Main config location**: `apps/web/src/app.css` contains the Tailwind configuration
- **Theme customization**: Custom colors, fonts, and utilities are defined in `app.css` using `@theme` blocks
- **Use `cn` utility**: Import from `$lib/utils` for conditional class application with proper merging
- **Prefer Tailwind classes**: Always use Tailwind utility classes instead of inline styles or custom CSS in `<style>` blocks when possible

### Vanilla HTML forms

Always prefer web standards, including using the `form` element for GET actions like filtering and pagination, and POST actions like updating a record.

**Use SvelteKit Remote Functions `form`** with Zod: the SvelteKit philosophy respects web standards and falls back to native form behaviour even when JavaScript is disabled, and also increase DevEx thanks to strong typing.

### TypeScript Coding Standards

- **Always use `type` over `interface`**: Prefer `type Props = {}` instead of `interface Props {}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnnify) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
