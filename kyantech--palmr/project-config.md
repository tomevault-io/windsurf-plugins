---
trigger: always_on
description: This file contains instructions for GitHub Copilot to help contributors work effectively with the Palmr codebase.
---

# GitHub Copilot Instructions for Palmr

This file contains instructions for GitHub Copilot to help contributors work effectively with the Palmr codebase.

## Project Overview

Palmr is a flexible and open-source alternative to file transfer services like WeTransfer and SendGB. It's built with:

- **Backend**: Fastify (Node.js) with TypeScript, SQLite database, and filesystem/S3 storage
- **Frontend**: Next.js 15 + React + TypeScript + Shadcn/ui
- **Documentation**: Next.js + Fumadocs + MDX
- **Package Manager**: pnpm (v10.6.0)
- **Monorepo Structure**: Three main apps (web, server, docs) in the `apps/` directory

## Architecture and Structure

### Monorepo Layout

```
apps/
├── docs/       # Documentation site (Next.js + Fumadocs)
├── server/     # Backend API (Fastify + TypeScript)
└── web/        # Frontend application (Next.js 15)
```

### Key Technologies

- **TypeScript**: Primary language for all applications
- **Database**: Prisma ORM with SQLite (optional S3-compatible storage)
- **Authentication**: Multiple OAuth providers (Google, GitHub, Discord, etc.)
- **Internationalization**: Multi-language support with translation scripts
- **Validation**: Husky pre-push hooks for linting and type checking

## Development Workflow

### Base Branch

Always create new branches from and submit PRs to the `next` branch, not `main`.

### Commit Convention

Use Conventional Commits format for all commits:

```
<type>(<scope>): <description>

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation changes
- test: Adding or updating tests
- refactor: Code refactoring
- style: Code formatting
- chore: Maintenance tasks
```

Examples:

- `feat(web): add user authentication system`
- `fix(api): resolve null pointer exception in user service`
- `docs: update installation instructions in README`
- `test(server): add unit tests for user validation`

### Code Quality Standards

1. **Linting**: All apps use ESLint. Run `pnpm lint` before committing
2. **Formatting**: Use Prettier for code formatting. Run `pnpm format`
3. **Type Checking**: Run `pnpm type-check` to validate TypeScript
4. **Validation**: Run `pnpm validate` to run both linting and type checking
5. **Pre-push Hook**: Automatically validates all apps before pushing

### Testing Changes

- Test incrementally during development
- Run validation locally before pushing: `pnpm validate` in each app directory
- Keep changes focused on a single issue or feature
- Review your work before committing

## Application-Specific Guidelines

### Web App (`apps/web/`)

- Framework: Next.js 15 with App Router
- Port: 3000 (development)
- Scripts:
  - `pnpm dev`: Start development server
  - `pnpm build`: Build for production
  - `pnpm validate`: Run linting and type checking
- Translations: Use Python scripts in `scripts/` directory
  - `pnpm translations:check`: Check translation status
  - `pnpm translations:sync`: Synchronize translations

### Server App (`apps/server/`)

- Framework: Fastify with TypeScript
- Port: 3333 (default)
- Scripts:
  - `pnpm dev`: Start development server with watch mode
  - `pnpm build`: Build TypeScript to JavaScript
  - `pnpm validate`: Run linting and type checking
  - `pnpm db:seed`: Seed database
- Database: Prisma ORM with SQLite

### Docs App (`apps/docs/`)

- Framework: Next.js with Fumadocs
- Port: 3001 (development)
- Content: MDX files in `content/docs/`
- Scripts:
  - `pnpm dev`: Start development server
  - `pnpm build`: Build documentation site
  - `pnpm validate`: Run linting and type checking

## Code Style and Best Practices

### General Guidelines

1. **Follow Style Guidelines**: Ensure code adheres to ESLint and Prettier configurations
2. **TypeScript First**: Always use TypeScript, avoid `any` types when possible
3. **Component Organization**: Keep components focused and single-purpose
4. **Error Handling**: Implement proper error handling and logging
5. **Comments**: Add comments only when necessary to explain complex logic
6. **Imports**: Use absolute imports where configured, keep imports organized

### API Development (Server)

- Use Fastify's schema validation for all routes
- Follow REST principles for endpoint design
- Implement proper authentication and authorization
- Handle errors gracefully with appropriate status codes
- Document API endpoints in the docs app

### Frontend Development (Web)

- Use React Server Components where appropriate
- Implement proper loading and error states
- Follow accessibility best practices (WCAG guidelines)
- Optimize performance (lazy loading, code splitting)
- Use Shadcn/ui components for consistent UI

### Documentation

- Write clear, concise documentation
- Include code examples where helpful
- Update documentation when changing functionality
- Use MDX features for interactive documentation
- Follow the existing documentation structure

## Translation and Internationalization

- All user-facing strings should be translatable
- Use the Next.js internationalization system
- Translation files are in `apps/web/messages/`
- Reference file: `en-US.json`
- Run `pnpm translations:check` to verify translations
- Mark untranslated strings with `[TO_TRANSLATE]` prefix

## Common Patterns

### Authentication Providers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyantech/Palmr](https://github.com/kyantech/Palmr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
