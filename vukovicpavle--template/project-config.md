---
trigger: always_on
description: You are an AI coding assistant working in a T3 Turbo monorepo. Follow these conventions strictly.
---

# AI Agent Context for T3 Turbo Monorepo

You are an AI coding assistant working in a T3 Turbo monorepo. Follow these conventions strictly.

## Project Structure

This is a Turborepo monorepo using:
- **Package Manager**: pnpm (workspace protocol)
- **Apps**: Next.js 15, Expo (React Native)
- **Packages**: Shared packages under `packages/`
- **Tooling**: Shared configs under `tooling/`

### Monorepo Layout
```
apps/
  ├─ nextjs/          # Next.js 15 app (web)
  └─ expo/            # Expo app (React Native)
packages/
  ├─ api/             # tRPC router (server-only)
  ├─ auth/            # Better Auth configuration
  ├─ db/              # Drizzle ORM schema & client
  ├─ store/           # Zustand state management
  ├─ ui/              # shadcn/ui components
  └─ validators/      # Zod schemas
tooling/
  ├─ eslint/          # Shared ESLint configs
  ├─ prettier/        # Shared Prettier config
  ├─ tailwind/        # Shared Tailwind configs
  └─ typescript/      # Shared tsconfig
```

## Code Style Rules

### TypeScript
- Use strict mode (already configured)
- Prefer type imports: `import type { X } from '...'`
- Use `satisfies` for type-safe object literals
- No `any` types - use `unknown` if needed
- Use `zod/v4` for validation: `import { z } from 'zod/v4'`

### Import Order (enforced by Prettier)
1. Type-only imports
2. React/Next.js/Expo
3. Third-party packages
4. `@acme/*` packages (types first, then values)
5. Local imports (`~/`, `../`, `./`)

### Naming Conventions
- **Files**: kebab-case for files, PascalCase for React components
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Types/Interfaces**: PascalCase
- **Functions**: camelCase, descriptive verbs

### React/Next.js Patterns
- Use Server Components by default
- Client Components only when needed (`'use client'`)
- Use `~/` alias for app-level imports
- Prefer `async` Server Components
- Use tRPC for API calls (type-safe)

### Error Handling
- Use Zod schemas for validation
- Throw errors with descriptive messages
- Use tRPC error codes appropriately
- Never expose sensitive data in errors

## Security Rules

### Environment Variables
- **NEVER** use `process.env` directly
- Always use validated env: `import { env } from '~/env'` or `@acme/env`
- Never commit `.env` files
- Never log secrets or API keys
- Use `.env.example` as template

### Secrets Handling
- No secrets in code
- No secrets in logs
- No PII in logs
- Use environment variables for all secrets

## File Organization

### Package Structure
Each package should have:
- `package.json` with proper workspace deps
- `tsconfig.json` extending `@acme/typescript-config/base`
- `eslint.config.js` using shared config
- `src/index.ts` as main entry point

### App Structure (Next.js)
- `src/app/` - App Router pages
- `src/components/` - React components
- `src/trpc/` - tRPC client/server setup
- `src/auth/` - Auth client/server utilities

### App Structure (Expo)
- `src/app/` - Expo Router pages
- `src/components/` - React Native components
- `src/utils/` - Utilities (api, auth, etc.)

## Dependencies

### Package Dependencies
- `@acme/api` - **dev dependency** in client apps (for types only)
- `@acme/api` - **production dependency** only in Next.js server
- Shared runtime code → create `@acme/shared` package

### Import Rules
- Use workspace protocol: `"@acme/ui": "workspace:*"`
- Prefer shared packages over duplication
- Keep dependencies minimal

## Testing & Quality

### Linting
- ESLint runs on all files
- TypeScript strict mode enabled
- Import order enforced
- No unused variables (prefix with `_` to ignore)

### Formatting
- Prettier with Tailwind plugin
- Auto-sort imports
- Run `pnpm format:fix` before committing

### Type Checking
- `pnpm typecheck` must pass
- No `@ts-ignore` or `@ts-expect-error` without comment

## Commit Messages

**MANDATORY**: All commit messages MUST follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. The agent MUST enforce this format for every commit.

### Commit Message Format

**Required Format**: `<type>(<scope>): <description>`

Where:
- `<type>`: One of the commit types below (REQUIRED)
- `<scope>`: Optional package/app name (e.g., `api`, `ui`, `nextjs`, `expo`, `cursor`)
- `<description>`: Clear, imperative-mood description (REQUIRED)

### Supported Commit Types

The following commit types are supported per the Conventional Commits specification:

- **`feat`**: A new feature (triggers minor version bump)
- **`fix`**: A bug fix (triggers patch version bump)
- **`docs`**: Documentation only changes
- **`style`**: Code style changes (formatting, missing semicolons, etc.) that do not affect code meaning
- **`refactor`**: Code restructuring that neither fixes a bug nor adds a feature
- **`perf`**: Performance improvements
- **`test`**: Adding or updating tests
- **`build`**: Changes to build system or external dependencies
- **`ci`**: Changes to CI configuration files and scripts
- **`chore`**: Other changes that don't modify src or test files (maintenance tasks)
- **`revert`**: Reverts a previous commit

### Enforcement Rules

1. **ALWAYS** format commit messages using the Conventional Commits format
2. **ALWAYS** use one of the supported commit types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vukovicpavle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
