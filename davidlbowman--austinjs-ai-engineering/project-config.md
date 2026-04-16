---
trigger: always_on
description: <!-- markdownlint-disable-file MD013 -->
---

<!-- markdownlint-disable-file MD013 -->
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-page static site about agentic coding, teaching people how to use multiple agents simultaneously with Claude Code and GitHub CLI. Built with Astro v5.11.0, React, Tailwind CSS v4, and shadcn/ui components.

## Common Commands

### Development

```bash
bun dev          # Start dev server at localhost:4321
bun build        # Build for production to ./dist/
bun preview      # Preview production build locally
```

### Code Quality

```bash
bun format       # Format code using Biome
bun lint         # Lint code using Biome  
bun check        # Run both format and lint checks
```

### Testing

```bash
bun typecheck    # Run TypeScript type checking
bun test:e2e     # Run Playwright E2E tests with text output
```

### Dependency Management

```bash
bun install      # Install dependencies
bun add <pkg>    # Add new dependency
```

## Architecture

### Technology Stack

- **Framework**: Astro with React integration
- **Styling**: Tailwind CSS v4 (using new Vite plugin approach)
- **Components**: shadcn/ui with Radix UI primitives
- **Language**: TypeScript with strict mode
- **Package Manager**: Bun
- **Code Quality**: Biome for formatting and linting
- **Testing**: Playwright for E2E tests (multi-browser: Chrome, Firefox, Edge, Mobile)

### Project Structure

- `src/pages/` - Astro pages (file-based routing)
- `src/components/ui/` - shadcn/ui components
- `src/lib/` - Utility functions
  - `utils.ts` - Includes `cn()` for className merging
- `src/styles/global.css` - Global styles with Tailwind directives and CSS variables
- `e2e/` - Playwright E2E tests (files must end with `-e2e.ts`)
- `public/` - Static assets served directly

### Key Configurations

- **TypeScript**: Path alias `@/*` maps to `./src/*`
- **Tailwind CSS**: Configured via astro.config.mjs using Vite plugin
- **shadcn/ui**: Configured in components.json with "new-york" style and CSS variables
- **Biome**: Auto-formatting on file modifications
- **Playwright**: Configured for textual output to enable test result reading
- **E2E Test Pattern**: Files in `e2e/` directory must end with `-e2e.ts`

## Frontend Development Patterns

### Core Philosophy

**React and Shadcn take priority in ALL cases**.

### Component Development

- Use shadcn/ui components directly - never modify them
- Add new components with: `bunx shadcn@latest add <component-name>`
- Use `cn()` utility for conditional classes:

  ```typescript
  import { cn } from "@/lib/utils"
  
  <div className={cn(
    "base-classes",
    condition && "conditional-classes"
  )} />
  ```

### CSS Architecture

- Tailwind CSS v4 with CSS custom properties for theming
- Dark mode support via CSS variables in src/styles/global.css
- Always use `cn()` for conditional classes
- Follow Tailwind's utility-first approach

## Testing Patterns

### E2E Testing with Playwright

**Configuration**: Tests output text format for Claude Code readability

**Test Structure**:

```typescript
import { test, expect } from '@playwright/test'

test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
  })

  test('should display expected content', async ({ page }) => {
    await expect(page.getByRole('heading', { name: 'Title' })).toBeVisible()
    await expect(page.getByRole('button', { name: 'Action' })).toBeVisible()
  })
})
```

**Best Practices**:

- Test user-visible behavior
- Use `getByRole()`, `getByText()` over CSS selectors
- Be specific with selectors when multiple elements exist
- Use `await expect()` for auto-retrying assertions

## MCP Tools Setup

The project is designed to demonstrate MCP (Model Context Protocol) tools integration. Future setup will include tools for enhanced development workflows.

## Development Workflow

### Quality Assurance Protocol

After completing EACH todo item, run these verification scripts:

```bash
bun typecheck    # Ensure TypeScript types are correct
bun check        # Run Biome format and lint checks
bun test:e2e     # Run Playwright E2E tests
```

**CRITICAL**: Never proceed to the next todo until all checks pass.

### Task Management

1. User assigns tasks
2. Create todos for each task
3. Complete one todo at a time
4. Run ALL verification scripts after each todo
5. Fix any issues before proceeding
6. Mark todo as completed only when all checks pass

## Important Notes

- **Static Site Focus**: This is a single-page site - avoid overcomplicating
- **Testing Output**: Playwright tests must use text reporter for readability
- **No Backend**: This is a static site with no server-side logic
- **Educational Purpose**: Code should be clear and demonstrate best practices for teaching
- **Quality Gates**: Every change must pass typecheck, biome check, and E2E tests

## UI/UX Strategy: Converting README to Interactive Single-Page App

### Design Philosophy

Transform the technical README content into an engaging, interactive experience using shadcn/ui components with creative layouts and micro-interactions. Each section should feel distinct while maintaining visual cohesion.

### Lessons Learned from Implementation

#### Component Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidlbowman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
