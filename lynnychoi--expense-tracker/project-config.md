---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Korean household expense tracking application - a single-page app for collaborative financial management between household members (max 7 per household). Built as a modern web application with real-time collaboration and Korean Won (₩) currency support.

## Commands

### Development
```bash
npm run dev          # Start development server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint - always run before commits
npm run typecheck    # Run TypeScript compiler check - ensure type safety
```

**Error Handling Protocol**: When any error occurs during development:
1. **Inspect and Document**: Analyze the error thoroughly and document what went wrong
2. **Present Options**: Provide multiple resolution approaches with trade-offs
3. **Recommend Best Option**: Suggest the optimal solution based on project context
4. **Wait for Approval**: Always wait for user confirmation before implementing fixes

### Visual Testing (Required for Frontend Changes)
After any frontend implementation, immediately run visual verification:
```bash
# Run Playwright in headed mode for visual inspection
npx playwright test --headed
```

### Git Workflow
Use [gitmoji.dev](http://gitmoji.dev) conventions for commit messages. Group related changes logically:
```bash
git add .
git commit -m "✨ feat: add Korean Won currency utilities"
git commit -m "♻️ refactor: organize transaction types in shared file"
git commit -m "🔧 chore: configure Shadcn/UI components"
```

### Shadcn/UI Components
```bash
npx shadcn@latest add [component]  # Add new Shadcn/UI components
```
Use `mcp__shadcn__getComponents` to discover available components and `mcp__shadcn__getComponent` for detailed implementation guidance before adding manually.

### Database (Requires Docker Desktop)
```bash
supabase start       # Start local Supabase (requires Docker)
supabase db reset    # Reset local database and apply migrations
supabase db push     # Push schema changes to remote
supabase status      # View connection details
```

## Architecture

### Tech Stack
- **Frontend**: Next.js 15 App Router, TypeScript, Tailwind CSS v4
- **UI**: Shadcn/UI components (configured for New York style)
- **Backend**: Supabase (PostgreSQL + Auth + Storage)  
- **Currency**: Korean Won (₩) stored as integers
- **AI**: OpenAI API integration planned

### Key Architectural Decisions

**Single Page Application**: All functionality lives on one main dashboard page (`src/app/page.tsx`) with modal dialogs for forms. This design prioritizes simplicity and real-time collaboration. After any change to the main dashboard, immediately test visual compliance across desktop, tablet, and mobile viewports using Playwright in headed mode.

**Korean Won Currency System**: All amounts stored as integers without decimals. Always use the established utilities in `src/lib/currency.ts` for consistency:
- `formatKRW(1234567)` → `"₩1,234,567"`
- `parseKRW("₩1,234,567")` → `1234567`
- `isValidKRWAmount()` for validation before storage

**Multi-household Architecture**: Users belong to one household at a time (max 7 members). Household creators have admin privileges. Data access controlled via Supabase Row Level Security.

**Component Structure**: Shadcn/UI components in `src/components/ui/` with path aliases configured (`@/components`, `@/lib`, etc.). Form validation with React Hook Form + Zod. Follow existing patterns when adding new components - maintain consistent import styles, use TypeScript strictly, and follow the established folder structure.

**Frontend Change Protocol**: IMMEDIATELY after implementing any UI change, perform visual verification:
1. Identify what changed - Review modified components/pages
2. Navigate to affected pages - Use `mcp__playwright__browser_navigate` to visit each changed view  
3. Capture screenshots - Take full page screenshots at desktop, tablet, mobile viewports
4. Verify design compliance - Compare against existing design patterns and UX best practices
5. Validate feature implementation - Ensure the change fulfills the user's specific request
6. Check acceptance criteria - Review any provided context files or requirements against PRD.md user stories
7. Check for errors - Run `mcp__playwright__browser_console_messages` and resolve if necessary
8. Update PRD status - Mark completed user stories in PRD.md, update specifications if requirements changed

**Error Resolution Process**: If errors are found during visual testing:
- Document the specific error with screenshots and console messages
- Identify root cause (CSS, JavaScript, API, database, etc.)
- Present 2-3 resolution options with pros/cons for each approach
- Recommend the best solution and wait for user approval before proceeding

**MCP Integration**: Leverage connected MCPs for development tasks - use Shadcn MCP for component discovery (`mcp__shadcn__getComponents`), GitHub MCP for repository management, and Vercel MCP for deployment operations rather than manual processes.

## Core Domain Models

### Transaction System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lynnychoi/expense-tracker](https://github.com/lynnychoi/expense-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
