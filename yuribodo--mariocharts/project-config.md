---
trigger: always_on
description: - Next.js pages live in `app/`; marketing blocks are in `components/site` and shared widgets in `components/ui`.
---

# Repository Guidelines

## Project Structure & Module Organization
- Next.js pages live in `app/`; marketing blocks are in `components/site` and shared widgets in `components/ui`.
- Published components stay in `src/components/**` with supporting hooks in `src/hooks` and shared utilities in `lib/`.
- CLI assets sit in `packages/cli`, registry metadata in `packages/registry`, and static files in `public/`; lean on the `@/...` aliases from `tsconfig.json`.

## DX & Design Principles
- Treat documentation as product: update `docs/`, snippets, and copy whenever APIs move.
- Keep onboarding frictionless—CLI templates must succeed on a clean workspace with all peers declared.
- Ship beautiful defaults that remain copy-paste friendly while revealing advanced switches only when needed.
- Guard “fail fast” behavior with actionable errors, strict TypeScript, and performance-conscious patterns.

## Build, Test, and Development Commands
- `npm run dev` boots the Turbopack site; `npm run dev:cli` watches template changes.
- `npm run build` runs the CLI build then the production Next.js build; use it as the release gate.
- `npm run lint` / `npm run lint:fix` apply the shared ESLint ruleset; pair with `npm run typecheck` before reviews.
- `npm run test` executes Jest suites, and `npm run storybook` opens Storybook for visual QA.

## Coding Style & Naming Conventions
- Prefer TypeScript functional components and React hooks for stateful logic.
- Follow Prettier/ESLint defaults (two spaces, double quotes) and let `lint:fix` resolve drift.
- Exported components and files use `PascalCase`, helpers use `camelCase`, and Tailwind strings should pass through `cn`.
- Co-locate prop types with the component and replace stray `console` statements with typed helpers or tests.

## Testing Guidelines
- Jest with `@testing-library/react` should cover interactions; keep specs beside implementations as `*.test.tsx`.
- Assert accessibility flows (focus, keyboard, ARIA) plus critical data branches for each chart variant.
- Mirror new scenarios in Storybook to document behavior and expose visual regressions early.

## Commit & Pull Request Guidelines
- Write imperative commit subjects similar to history (`Add LineChart Component`, `Feat/bar chart`) with optional `(#123)` tags.
- Run lint, typecheck, and test before pushing, then report results inside the PR description.
- PRs must describe intent, link issues, and include screenshots or clips for UI changes.

## Storybook & Docs Workflow
- When component APIs shift, update `docs/`, Storybook stories, and CLI templates, then validate via `npm run dev:cli`.
- Keep `mario-charts.json` and `components.json` aligned with new exports so scaffolding metadata stays current.

## Specialized Agents
- Reference `agents/bar-chart-architect.md` when implementing or optimizing complex bar chart systems; it captures the dedicated persona, architecture standards, and workflow for high-stakes chart work.

## Mario Charts - React Dashboard Component Library

### Overview

Mario Charts is a modern React component library focused on charts and dashboards. Prioritizes beautiful visuals out-of-the-box, ease of use, and excellent developer experience through copy-and-paste components with zero lock-in.

### DX Philosophy - Developer Experience First

#### Core DX Principles

**"Documentation IS the product"** - Every interaction point is carefully crafted for developer success. We believe that the quality of developer experience directly correlates with adoption and satisfaction.

**Zero Friction Onboarding**
- No signup required to start
- No credit card or demo calls
- 5-minute guarantee from install to working chart
- First impression must be flawless

**Fail Fast, Succeed Faster**
- Clear error messages with actionable solutions
- Comprehensive TypeScript support prevents runtime errors
- Extensive testing ensures reliability
- Performance-first approach prevents scalability issues

#### Design Philosophy

**Beautiful Defaults, Infinite Customization**
- Components look professional out-of-the-box
- Zero configuration required for basic usage
- Deep customization available when needed
- Consistent design language across all components

**Copy-Paste Philosophy**
- Full ownership of component code
- No black box abstractions
- Easy to modify and extend
- AI-readable code structure

**Progressive Disclosure**
- Simple API for basic usage
- Advanced features available without complexity
- Clear upgrade paths for growing needs
- Modular architecture allows selective adoption

### Tech Stack

#### Core Dependencies
- **React** 18+ - Base for reusable components
- **Tailwind CSS** - Design system and styling
- **Radix UI** - Accessible primitives for interactive components
- **Recharts** (peer dependency) - Chart engine foundation
- **date-fns** - Date manipulation for filters
- **Framer Motion** - Advanced animations and micro-interactions

#### Distribution
- **shadcn/ui CLI system** - Copy-and-paste distribution for maximum flexibility
- **Custom registry** - Versioned and maintained components
- **Zero lock-in** - Users copy code directly to their project

#### Project Structure
```
mario-charts/
├── src/
│   ├── components/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuribodo/mariocharts](https://github.com/yuribodo/mariocharts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
