---
trigger: always_on
description: SupportSpark is a support network platform helping people share updates with their trusted support network during difficult times. Built with React 19, Express 5, TypeScript, and deployed on Windows 11 + IIS.
---

# GitHub Copilot Instructions for SupportSpark

## Project Overview

SupportSpark is a support network platform helping people share updates with their trusted support network during difficult times. Built with React 19, Express 5, TypeScript, and deployed on Windows 11 + IIS.

## Governance

**ALL CODE AND DOCUMENTATION MUST COMPLY WITH THE PROJECT CONSTITUTION**

📘 **Constitution Location**: [.documentation/memory/constitution.md](../.documentation/memory/constitution.md)

The constitution defines 10 core principles (Type Safety, Testing, UI Components, Security, API Contracts, State Management, Code Style, Data Storage, Deployment, and Simplicity First). Review it before making any architectural decisions.

## Documentation Standards

### Generated Documentation Location

**ALL Copilot-generated documentation MUST be saved under `/.documentation/copilot/`**

**Naming Convention**: `session-{YYYY-MM-DD}/`

Examples:
- `/.documentation/copilot/session-2026-02-03/architecture-review.md`
- `/.documentation/copilot/session-2026-02-03/refactoring-plan.md`
- `/.documentation/copilot/session-2026-02-03/bug-analysis.md`

**Folder Structure**:
```
.documentation/
├── memory/              # Long-term project memory (constitution, etc.)
├── domain/              # Architectural and domain documentation
├── specs/               # Feature specifications (spec-driven development)
├── scripts/             # PowerShell automation scripts
│   └── powershell/      # PowerShell-specific scripts
├── templates/           # Templates for specs, plans, tasks, etc.
└── copilot/             # Copilot-generated session documentation
    └── session-{date}/  # Date-based session folders
```

### Documentation Types

- **Architecture decisions**: `/.documentation/copilot/session-{date}/architecture-*.md`
- **Feature specifications**: `/.documentation/copilot/session-{date}/spec-*.md`
- **Bug investigations**: `/.documentation/copilot/session-{date}/bug-*.md`
- **Refactoring plans**: `/.documentation/copilot/session-{date}/refactor-*.md`
- **Code reviews**: `/.documentation/copilot/session-{date}/review-*.md`

## Technology Stack

### Frontend
- **Framework**: React 19 (functional components only)
- **Build Tool**: Vite
- **UI Library**: shadcn/ui + Radix UI
- **Styling**: Tailwind CSS
- **Routing**: Wouter
- **State**: TanStack React Query (server state), useState/useReducer (client state)
- **Animations**: Framer Motion

### Backend
- **Runtime**: Node.js with Express 5
- **Language**: TypeScript (strict mode)
- **Auth**: Passport.js + express-session
- **Validation**: Zod (all schemas in `shared/schema.ts`)

### Development
- **Package Manager**: npm
- **Testing**: Vitest (preferred) or Jest
- **Linting**: ESLint + Prettier
- **Build Output**: CommonJS (`.cjs`) for iisnode compatibility

## Code Generation Rules

### Type Safety (NON-NEGOTIABLE)
- ✅ Always use TypeScript with `strict: true`
- ✅ Define all schemas with Zod in `shared/schema.ts`
- ✅ Use path aliases: `@/` (client), `@shared/` (shared)
- ❌ Never use `any` without explicit justification in comments
- ✅ Validate all API inputs/outputs at runtime with Zod

### UI Components (NON-NEGOTIABLE)
- ✅ Use shadcn/ui components from `@/components/ui/`
- ✅ Extend Radix UI primitives when needed
- ✅ Use Tailwind CSS for all styling (no inline styles or CSS modules)
- ✅ Suggest `npx shadcn-ui@latest add <component>` for missing components
- ❌ Never create custom UI primitives that duplicate shadcn/ui

### Testing (NON-NEGOTIABLE)
- ✅ Generate test files alongside feature implementation
- ✅ Name test files `*.test.ts` or `*.test.tsx`
- ✅ Write integration tests for all API routes
- ✅ Write unit tests for React hooks
- ✅ Use Vitest syntax (compatible with Jest)

### API Development (RECOMMENDED)
- ✅ Define stable API routes in `shared/routes.ts`
- ✅ Use standardized error schemas from `shared/schema.ts`
- ✅ During prototyping, simple endpoints without contracts are acceptable
- ✅ Add contracts when endpoints stabilize or have multiple clients

### Security (NON-NEGOTIABLE)
- ✅ Hash passwords with bcrypt (minimum 10 rounds)
- ✅ Use environment variables for secrets (never hardcode)
- ✅ Implement rate limiting before public access
- ✅ Add CSRF protection for production deployments
- ✅ Validate file uploads (type and size)

### Code Style (NON-NEGOTIABLE)
- ✅ Follow existing ESLint + Prettier configuration
- ✅ Import order: external deps → `@shared/` → `@/` → relative
- ✅ Use descriptive variable names
- ✅ Add JSDoc comments for complex functions
- ✅ Keep functions small and focused

### Simplicity First (NON-NEGOTIABLE)
- ✅ Start with the simplest implementation
- ✅ Use JSON file storage before databases
- ✅ Use React's useState before Redux/Zustand
- ✅ Write inline code before extracting utilities
- ❌ Don't add features until they're actually needed (YAGNI)
- ❌ Don't prematurely optimize
- ❌ Don't create abstractions for single-use code

## File Structure Patterns

### Client Structure
```
client/src/
├── components/          # React components
│   ├── ui/             # shadcn/ui primitives (DO NOT EDIT)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markhazleton/SupportSpark](https://github.com/markhazleton/SupportSpark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
