---
trigger: always_on
description: This is **Sitecore JavaScript Services (JSS)** — a TypeScript/JavaScript SDK for building modern web applications with Sitecore XM Cloud and on-premises. The project provides core layout/GraphQL functionality, React/Angular/Vue/Next.js integrations, and CLI tools for scaffolding applications.
---

# Claude Code Agent Guide for Sitecore JavaScript Services (JSS)

## Project Overview

This is **Sitecore JavaScript Services (JSS)** — a TypeScript/JavaScript SDK for building modern web applications with Sitecore XM Cloud and on-premises. The project provides core layout/GraphQL functionality, React/Angular/Vue/Next.js integrations, and CLI tools for scaffolding applications.

### Tech Stack
- **Language**: TypeScript (Node LTS)
- **Runtime**: Node LTS (>=24)
- **Build**: `tsc` → `dist/`, templates built via `scripts/build-templates.ts` in create-sitecore-jss
- **Testing**: Mocha + Sinon + Chai, coverage via `nyc`
- **Lint/format**: ESLint + Prettier
- **Package Manager**: Yarn 4.12.0

### Core Packages
- `@sitecore-jss/sitecore-jss` - Core SDK (layout, GraphQL, i18n, tracking, editing)
- `@sitecore-jss/sitecore-jss-react` - React components (Text, Image, Link, Placeholder)
- `@sitecore-jss/sitecore-jss-nextjs` - Next.js integration and middleware
- `@sitecore-jss/sitecore-jss-angular` - Angular integration
- `@sitecore-jss/sitecore-jss-vue` - Vue integration
- `@sitecore-jss/sitecore-jss-cli` - CLI (jss deploy, jss setup)
- `create-sitecore-jss` - Scaffolding CLI and templates

## Repository Structure

```
jss/
├── packages/
│   ├── create-sitecore-jss/    # Scaffolding CLI & templates
│   ├── sitecore-jss/           # Core SDK (layout, GraphQL, i18n, tracking)
│   ├── sitecore-jss-react/     # React components
│   ├── sitecore-jss-nextjs/    # Next.js integration
│   ├── sitecore-jss-angular/   # Angular integration
│   ├── sitecore-jss-vue/      # Vue integration
│   ├── sitecore-jss-cli/      # CLI tools
│   ├── sitecore-jss-forms/    # Forms
│   ├── sitecore-jss-proxy/    # Proxy / rendering host
│   └── ...                     # Other packages
├── samples/                    # Example applications (generated)
├── scripts/                    # scaffold-samples.js, lint-samples.js, samples.json
└── ref-docs/                   # Generated API docs (yarn generate-docs)
```

### Key Directories
- **Sources**: `src/**` in each package
- **Never edit**: `dist/**` (compiled output)
- **Templates**: `packages/create-sitecore-jss/src/templates/` — nextjs, angular, vue, react, etc. Use `.env` with EJS tokens for env values
- **Initializers**: `packages/create-sitecore-jss/src/initializers/` — each exposes `init(args)`, driven by `initRunner(initializers, args)`
- **Common processes**: `packages/create-sitecore-jss/src/common/` — transform, install, lint utilities

## Code Style

### Vibe-Coding Principles

Core Philosophy:
- Write clean, modular, idiomatic code
- Prefer declarative over imperative patterns
- Make code readable and self-documenting
- TypeScript-first development approach

Code Organization:
- Use Node LTS
- Export public types at module boundaries
- Prefer pure functions and thin wrappers
- No top-level side effects (except CLI entry)
- Modular architecture with clear separation of concerns

### Code Quality Standards

TypeScript Usage:
- Enable strict mode in all projects
- Prefer explicit types over `any`
- Use discriminated unions for complex state
- Export types at module boundaries for reusability

Functional Programming:
- Prefer pure functions where possible
- Use immutable data patterns
- Avoid side effects in business logic
- Compose small, focused functions

Readability:
- Use descriptive variable and function names
- Keep functions small and focused (single responsibility)
- Add comments for complex business logic only
- Prefer self-documenting code over extensive comments

### Error Handling

Error Strategy:
- Fail fast with clear, actionable messages
- Propagate child-process errors with context
- Use custom error classes for domain-specific errors
- Handle edge cases explicitly with guard clauses

Security:
- Sanitize user inputs
- Validate data at boundaries
- Never log sensitive information
- Use environment variables for configuration

### Development Workflow

Logging:
- Clear phases and results
- Support silent flag if available
- Use appropriate log levels (debug, info, warn, error)
- Include context in error messages

Imports:
- Relative for local modules
- Never import from `dist/`
- Group imports logically (external, internal, relative)
- Use barrel exports (index.ts) for clean APIs

Lint and Format:
- Keep ESLint + Prettier passing at all times
- Follow configured style rules consistently
- Use automated formatting on save
- Address linting warnings promptly

## JavaScript/TypeScript Rules

### Naming Conventions

Variables and Functions:
- Use camelCase: `getLayoutData()`, `isConnected`, `routeData`
- Boolean variables: prefix with `is`, `has`, `can`, `should`
- Event handlers: prefix with `handle` or `on`: `handleClick`, `onSubmit`

Components (React):
- Use PascalCase: `Placeholder`, `Text`, `Image`, `Link`
- File names match component names: `Placeholder.tsx`

Constants:
- Use UPPER_SNAKE_CASE: `JSS_MODE`, `FETCH_WITH`, `GRAPH_QL_ENDPOINT`
- Export at module level when shared

Directories:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sitecore/jss](https://github.com/Sitecore/jss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
