---
trigger: always_on
description: - **Monorepo**: Turborepo-managed, with `apps/` (main: Next.js web app) and `packages/` (shared code: `common`, `shared`, `ai`, `ui`, etc.).
---


# AGENT.md - Development Guidelines

## Tech Stack & Project Overview

- **Monorepo**: Turborepo-managed, with `apps/` (main: Next.js web app) and `packages/` (shared code: `common`, `shared`, `ai`, `ui`, etc.).
- **Core Technologies**: Next.js 15 (App Router), React 19.0.0, TypeScript, Tailwind CSS, shadcn/ui, Zustand, Drizzle ORM (Neon PostgreSQL), Better-Auth, Framer Motion, Lucide icons.
- **AI/Agents**: Agentic Graph System in `packages/ai/` (supports OpenAI, Anthropic, Google, Groq, etc.).
- **Best Practices**: Use environment variables, enums for string keys, named exports, shadcn/ui for UI, Bun for all scripts, and document changes in `memory-bank/`.

## Package Management

- Use `bun` instead of `npm` for all operations

## Code Style

- Make sure no string in #codebase, use enum pattern.
- Don't hard code values in the codebase.
- Use environment variables for configuration (e.g., API keys, product IDs, ADMIN_USER_IDS for Better-Auth admin access - supports comma-separated user IDs)
- Use centralize enum for custom reusable keys.
- 4-space indentation, single quotes, 100 char line length
- PascalCase components, camelCase hooks/utils, kebab-case files
- Named exports preferred over default exports
- Use oxlint for fast comprehensive linting (run `bun run lint`)
- Use dprint for code formatting (run `bun run fmt`)
- Use Prettier for markdown files only (run `bun run format`)

## UI/UX Design Principles

- **Minimal Design**: Follow shadcn/ui principles with clean, minimal aesthetics
- **No Colors**: Use only black/white/muted colors (avoid gradients, bright colors)
- **Minimal Icons**: Reduce icon usage to essentials only
- **Clean Typography**: Rely on typography hierarchy over visual decorations
- **Neutral Palette**: Use `text-muted-foreground`, `bg-muted`, standard shadcn colors
- **Simple Interactions**: Avoid flashy animations or complex visual effects

## Development Workflow

- Make sure you DO NOT CREATE ANY debug and test FILES IN ./ ROOT DIRECTORY. Only use files in /temps or apps/web/app/tests/ or /scripts.
- Make sure you run `bun dev` and check the app console to see if there are any errors before starting to work on anothers task. fix it first.
- Plan first, then implement. For substantial changes, request maintainer feedback on the plan before coding.
- **REQUIRED**: Run `bun run fmt` to auto-fix formatting issues with dprint
- **NEVER commit changes yourself** - DO NOT execute `git commit` unless you have my approval
- Run `bun run lint` (oxlint) for comprehensive error checking
- Run `bun run build` to verify compilation before major changes
- Test core functionality after significant changes

## Deployment

⚠️ **CRITICAL DEPLOYMENT POLICY** ⚠️
**DO NOT DEPLOY TO PRODUCTION FLY.IO WITHOUT EXPLICIT USER APPROVAL**

- NEVER run `./deploy-fly.sh` without user permission
- Always ask for approval before any production deployment
- This applies to ALL deployment commands and scripts

- **Production Deployment**: Use `./deploy-fly.sh` to deploy to Fly.io (ONLY WITH USER APPROVAL)
  - **Interactive**: `./deploy-fly.sh` (prompts for version bump type)
  - **Automated**: `./deploy-fly.sh --auto --version patch` (patch/minor/major)
  - **Features**: Auto-commit, semantic versioning, git tagging, Fly.io deployment
  - **App URL**: https://vtchat.io.vn (primary) / https://vtchat.fly.dev (backup)
  - Script handles: git status checks, version tagging, pushing to remote, Fly.io deployment

### Git Hooks

- **Manual fixes**: Run `bun run fmt` and `bun run fmt:check` for comprehensive formatting fixes with dprint
- **Philosophy**: Encourage good practices without blocking development flow

## Tech Stack

- Next.js 15 with App Router, React 19.0.0, TypeScript, Tailwind CSS
- Zustand for state, Drizzle ORM for DB, Better Auth for authentication
- Custom bot detection with Better-Auth plugin using isbot library
- Framer Motion for animations, Radix UI components
- Shadcn/ui for UI components, Lucide icons, clsx for classnames
- Payment integration with Creem.io

## Architecture

- Turborepo monorepo: `apps/` and `packages/`
- `@repo/common` - components/hooks, `@repo/shared` - types/utils
- Use `'use client'` for client components

## Domain Knowledge

- Chat application with AI models (OpenAI, Anthropic, etc.)
- Subscription tiers:VT offers free tier, and with VT+ focusing only on 3 exclusive research capabilities: Deep Research, Pro Search.
- MCP integration for external tools
- Use promptBoost tools to enhance prompt quality
- You can use playwright MCP tool to test web components integration

## Testing

- use ChatMode.ChatMode.GEMINI_3_FLASH_LITE to test instead GEMINI_3_PRO because cost.
- Test files should be in `apps/web/app/tests/`. Example: `./test-vt-plus-only.js` should be moved to `apps/web/app/tests/test-vt-plus-only.js`
- Every implemented feature should have a test case to maintain quality
- Every unit test should cover critical paths and edge cases
- Use `vitest` for testing, with `@testing-library/react` for React components.
- Run tests regularly to ensure code quality
- Use `@testing-library/jest-dom/vitest` for custom matchers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinhnx/vtchat](https://github.com/vinhnx/vtchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
