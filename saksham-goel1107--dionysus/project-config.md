---
trigger: always_on
description: Dionysus is an enterprise-grade GitHub analytics and collaboration SaaS platform built with Next.js 14, TypeScript, tRPC, Prisma, and PostgreSQL. It provides AI-powered code analysis, meeting transcription, team collaboration, and comprehensive repository insights.
---

# Dionysus: Enterprise GitHub Analytics & Collaboration Platform

Dionysus is an enterprise-grade GitHub analytics and collaboration SaaS platform built with Next.js 14, TypeScript, tRPC, Prisma, and PostgreSQL. It provides AI-powered code analysis, meeting transcription, team collaboration, and comprehensive repository insights.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Setup
- **Node.js requirement**: Node.js >=20.0.0, npm >=9.0.0
- **Install dependencies**: `npm install` -- installs successfully but may fail on Prisma generation due to network restrictions
- **Alternative installation**: `npm install --ignore-scripts` -- use this if postinstall hook fails
- **Environment setup**: The `.env.example` file is encrypted. Use `SKIP_ENV_VALIDATION=true` for development/testing
- **Prisma client**: May need manual generation with dummy DATABASE_URL if network access to Prisma binaries is restricted

### Build and Development Commands
- **Development server**:
  ```bash
  SKIP_ENV_VALIDATION=true NODE_OPTIONS='--max-old-space-size=4096' npx next dev --turbo
  ```
  - Starts successfully in ~5 seconds on port 3000
  - Requires memory optimization flag (4GB)
  - Works without full environment setup
  - Uses Turbopack for fast development

- **Production build**:
  ```bash
  SKIP_ENV_VALIDATION=true npm run build
  ```
  - **NEVER CANCEL**: Build takes 3-4 minutes to complete. Set timeout to 10+ minutes.
  - Requires `SKIP_ENV_VALIDATION=true` or proper environment variables
  - Currently fails due to TypeScript strict mode errors (41 errors in 15 files)
  - Compilation phase completes successfully before TypeScript validation

- **Type checking**:
  ```bash
  npm run typecheck
  ```
  - Takes ~30 seconds
  - Currently has 41 TypeScript errors related to implicit 'any' types
  - Does not require environment variables

### Linting and Formatting
- **ESLint check**:
  ```bash
  SKIP_ENV_VALIDATION=true npx next lint
  ```
  - **OR** `npx eslint src --ext .ts,.tsx --max-warnings 0`
  - Passes successfully with no warnings/errors
  - Takes ~10-15 seconds

- **Prettier formatting**:
  ```bash
  npm run format:check   # Check formatting
  npm run format:write   # Apply formatting
  ```
  - Both commands work perfectly
  - Uses Tailwind CSS plugin for class sorting
  - Takes ~5-10 seconds

### Database Operations
- **Important**: Prisma client generation may fail due to network restrictions to `binaries.prisma.sh`
- **Database URL required**: PostgreSQL with vector extensions support
- **Schema location**: `prisma/schema.prisma`
- **Commands available**: `db:generate`, `db:migrate`, `db:push`, `db:studio`
- **Note**: Database commands will fail without proper DATABASE_URL

### Testing
- **Test suite**: `npm test` -- Currently just exits with code 0 (no actual tests implemented)
- **No test infrastructure**: No Jest, Vitest, or other testing frameworks configured

## Validation

### Manual Validation Scenarios
- **ALWAYS test the development server startup**: Use the dev command above and verify it starts on port 3000
- **Test basic routing**: After starting dev server, check `curl -I http://localhost:3000` returns 200 OK
- **Validate linting passes**: Run both ESLint commands to ensure code quality
- **Check formatting**: Run format:check to ensure code style compliance
- **Build validation**: Attempt build with `SKIP_ENV_VALIDATION=true` and verify it progresses through compilation

### Environment Limitations
- **Doppler dependency**: Production scripts use `doppler run --` but this fails in development environments
- **Encrypted environment**: `.env.example` is encrypted and requires decryption key from repository owner
- **Network restrictions**: Prisma binary downloads and some external services may be blocked
- **Sentry warnings**: Expected warnings about missing auth tokens during build (non-blocking)

### CI/CD Integration
- **GitHub Actions**: Multiple workflows including ESLint, CodeQL, test, audit
- **Husky hooks**: Pre-commit and pre-push hooks enforce code quality
- **Branch naming**: Required pattern: `feature/*`, `fix/*`, `chore/*`, `refactor/*`, `test/*`, `hotfix/*`, `release/*`
- **Commit format**: Uses conventional commits (feat:, fix:, docs:, etc.)

## Common Tasks

### Repository Structure Overview
```
dionysus/
├── src/app/           # Next.js App Router pages and API routes
├── src/components/    # Reusable React components and UI library
├── src/lib/          # Utility functions, API clients, external integrations
├── src/server/       # tRPC server setup and database utilities
├── src/trpc/         # tRPC client setup and React Query integration
├── prisma/           # Database schema and migrations
├── public/           # Static assets and media files
└── .github/          # CI/CD workflows and issue templates
```

### Key Technologies
- **Frontend**: Next.js 15, React 19, TypeScript, TailwindCSS, shadcn/ui

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Saksham-Goel1107/Dionysus](https://github.com/Saksham-Goel1107/Dionysus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
