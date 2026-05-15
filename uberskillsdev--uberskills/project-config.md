---
trigger: always_on
description: > Context file for Claude Code when working on this codebase.
---

# CLAUDE.md -- uberSKILLS

> Context file for Claude Code when working on this codebase.

## Project Overview

uberSKILLS is an open-source web application for designing, testing, and deploying Claude Code Agent Skills. It provides a visual editor with AI-assisted creation, a multi-model testing sandbox, and one-click deployment to `~/.claude/skills/`.

The core workflow: **Create** a skill (manually or via AI chat) → **Edit** metadata, instructions, and files → **Test** with streaming AI responses and metrics → **Export** as zip or **Deploy** to the local filesystem.

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Monorepo | Turborepo + pnpm | turbo ^2, pnpm 9.15.4 |
| Framework | Next.js (App Router) | ^15 |
| UI | React + shadcn/ui + Tailwind CSS v4 | react ^19 |
| Language | TypeScript (strict) | ^5.7 |
| Database | SQLite (better-sqlite3) / Turso (@libsql/client) via Drizzle ORM | -- |
| AI | Vercel AI SDK + OpenRouter (@openrouter/ai-sdk-provider) | -- |
| Logging | Pino + pino-pretty | pino ^10, pino-pretty ^13 |
| Linting | Biome | ^2.4.5 |
| Testing | Vitest (unit) + Playwright (E2E) | -- |
| Fonts | Geist Sans + Geist Mono (via next/font) | -- |

## Monorepo Structure

```
uberskills/
├── apps/
│   ├── web/                    # Next.js 15 — Skills editor app (port 3000)
│   │   ├── app/                # Routes and API handlers (dashboard at /)
│   │   ├── components/         # React components
│   │   ├── e2e/                # Playwright E2E tests
│   │   ├── lib/                # Utilities and constants
│   │   └── styles/             # Global CSS (imports @uberskills/ui/globals.css)
│   └── landing/                # Next.js 15 — Static landing page (port 3001)
│       ├── app/                # Single-page layout + page
│       ├── components/landing/ # Landing page sections
│       ├── hooks/              # useInView hook
│       ├── lib/                # Utilities
│       └── styles/             # Global CSS (imports @uberskills/ui/globals.css)
├── packages/
│   ├── types/                  # @uberskills/types -- shared TypeScript interfaces
│   ├── db/                     # @uberskills/db -- Drizzle schema, queries, crypto
│   ├── skill-engine/           # @uberskills/skill-engine -- parser, validator, generator, importer, exporter
│   └── ui/                     # @uberskills/ui -- shadcn/ui components, ThemeProvider, design tokens
├── docs/                       # Project documentation
├── Dockerfile                  # Multi-stage Docker build
├── docker-compose.yml          # Docker Compose service definition
├── CONTRIBUTING.md             # Contribution guidelines
└── LICENSE                     # MIT license
```

### Package Responsibilities

- **@uberskills/types**: All shared interfaces (`Skill`, `SkillFrontmatter`, `SkillFile`, `SkillVersion`, `TestRun`, `AppSettings`, `ValidationError`) and enum types (`SkillStatus`, `TestRunStatus`, `FileType`, `Theme`).
- **@uberskills/db**: Drizzle ORM schema for 5 SQLite tables (`skills`, `skill_files`, `skill_versions`, `test_runs`, `settings`), typed query functions, database client with auto-migration, and AES-256-GCM encryption for API key storage.
- **@uberskills/skill-engine**: SKILL.md parser (YAML frontmatter + markdown body), validator (field presence/length/regex), generator (data → SKILL.md string), argument substitution (`$VARIABLE_NAME` placeholders), importer (zip/directory → parsed skills), exporter (skill → zip/filesystem).
- **@uberskills/ui**: Shared shadcn/ui components (Button, Input, Card, Dialog, Badge, etc.), `ThemeProvider`, and the full design system CSS (`globals.css` with color tokens, typography, animations).

## Common Commands

```bash
pnpm install          # Install all dependencies
pnpm dev              # Start all dev servers (web :3000, landing :3001)
pnpm dev:web          # Start only the editor app (port 3000)
pnpm dev:landing      # Start only the landing page (port 3001)
pnpm build            # Production build all packages + apps
pnpm lint             # Lint all files with Biome
pnpm lint:fix         # Auto-fix lintable issues
pnpm format           # Format all files with Biome
pnpm typecheck        # TypeScript type checking across all packages
pnpm test             # Run Vitest unit tests
pnpm run test:e2e     # Run Playwright E2E tests
pnpm run db:migrate   # Run database migrations
```

## Coding Conventions

### TypeScript

- **Strict mode** is enabled globally: `strictNullChecks`, `noUncheckedIndexedAccess`, `isolatedModules`.
- Do not use `any` -- Biome enforces `noExplicitAny` as an error.
- Unused variables and imports are errors.
- Use `const` over `let` where possible (enforced by Biome).

### Formatting (Biome)

- **Indent**: 2 spaces
- **Line width**: 100 characters
- **Quotes**: double quotes (`"`)
- **Semicolons**: always
- **Trailing commas**: always (except JSON)

### File Naming

- **Files**: `kebab-case.ts`, `kebab-case.tsx`
- **React components**: `PascalCase` exports in `kebab-case.tsx` files
- **Test files**: `__tests__/module-name.test.ts` colocated in `src/`
- **E2E tests**: `apps/web/e2e/*.spec.ts`

### Imports

- Use workspace package names: `import { Skill } from "@uberskills/types";`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uberskillsdev/uberSKILLS](https://github.com/uberskillsdev/uberSKILLS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
