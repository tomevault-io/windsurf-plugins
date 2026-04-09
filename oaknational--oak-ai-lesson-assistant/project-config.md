---
trigger: always_on
description: You are an expert in TypeScript, React, Next.js, tRPC, Zustand, Tailwind CSS, and Turborepo monorepo architecture.
---

You are an expert in TypeScript, React, Next.js, tRPC, Zustand, Tailwind CSS, and Turborepo monorepo architecture.

This is the monorepo for Aila, Oak National Academy's AI assistant.

TECHNOLOGY STACK

Frontend:

- Framework: Next.js with App Router
- Language: TypeScript (strict mode)
- UI Components: Primarily custom components from Oak's shared component library
- Styling: Tailwind CSS
- State Management:
  - Zustand for complex state (chat, lesson plan, moderation)
  - React hooks for component state
  - tRPC for data fetching and mutations

Backend:

- Architecture: tRPC API endpoints for type-safe client-server communication
- Language: TypeScript (strict mode)
- Database: Prisma ORM with PostgreSQL
- AI Integration: OpenAI, Langchain, Claude
- Authentication: Clerk

Deployment:

- Vercel

CODING STYLE

General Principles:

- Write concise, technical TypeScript code
- Use functional, declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names (e.g., isLoading, hasError)
- Follow "use client" pattern correctly for Next.js App Router

Formatting:

- Use 2-space indentation
- 80 character line width limit
- Double quotes for strings
- Include trailing commas
- Sort imports: React → Third-party → Project → Relative

TypeScript Usage:

- Use strict typing with noUncheckedIndexedAccess
- Prefer interfaces over types for object shapes
- Use type for complex types or unions
- Avoid enums; use string literal unions or const objects instead
- Always handle Promise rejections with @typescript-eslint/no-floating-promises rule
- Use consistent type imports (@typescript-eslint/consistent-type-imports)

Component Structure:

- Use functional components with React hooks
- Prefer named exports for components
- Add "use client" directive to client components
- Structure files: exported component, subcomponents, helpers, types
- Keep components focused on a single responsibility

tRPC Implementation:

- Use type-safe end-to-end communication
- Implement proper error handling in tRPC procedures
- Follow the project's router organization
- Use appropriate middleware (auth, rate limiting)

Error Handling:

- Handle errors and edge cases early in functions with guard clauses
- Use early returns for error conditions to avoid nesting
- Implement proper error logging and user-friendly messages
- Always handle Promise rejections using await or .catch()

Testing:

- Write Jest unit tests for utility functions and hooks
- Use Playwright for E2E testing
- Follow the project's test patterns and mocking strategies

MONOREPO ORGANIZATION

- Follow the established package structure:
  - apps/nextjs: Main Next.js application
  - packages/aila: Core AI assistant functionality
  - packages/api: tRPC API endpoints and routers
  - packages/core: Shared business logic and models
  - packages/db: Prisma schema and database access
  - packages/exports: Document export functionality
  - packages/logger: Structured logging utilities

NAMING CONVENTIONS

- PascalCase for component names, type names, and interfaces
- camelCase for variables, functions, and properties
- Use lowercase with dashes for directory names (kebab-case)
- For hooks, always prefix with "use" (e.g., useAnalytics)
- For Zustand store files, follow the established pattern with selectors and types

PERFORMANCE OPTIMIZATION

- Minimize client-side JavaScript with strategic "use client" usage
- Use React Server Components where possible
- Implement proper data fetching strategies with tRPC
- Use React.memo() for expensive renders when needed
- Dynamic imports for code splitting
- Optimize images with Next.js Image component
- Consider Web Vitals in UI implementations

SECURITY BEST PRACTICES

- Never expose sensitive data in client-side code
- Validate all user inputs, both client and server-side
- Implement proper authentication and authorization checks with Clerk
- Use tRPC's type safety for data validation
- Protect against rate limiting in sensitive endpoints

<cursor-tools Integration>
# Instructions
Use the following commands to get AI assistance:

**Web Search:**
`cursor-tools web "<your question>"` - Get answers from the web using Perplexity AI (e.g., `cursor-tools web "latest weather in London"`)
when using web for complex queries suggest writing the output to a file somewhere like local-research/<query summary>.md.

**Repository Context:**
`cursor-tools repo "<your question>"` - Get context-aware answers about this repository using Google Gemini (e.g., `cursor-tools repo "explain authentication flow"`)

**Documentation Generation:**
`cursor-tools doc [options]` - Generate comprehensive documentation for this repository (e.g., `cursor-tools doc --output docs.md`)
when using doc for remote repos suggest writing the output to a file somewhere like local-docs/<repo-name>.md.

**GitHub Information:**
`cursor-tools github pr [number]` - Get the last 10 PRs, or a specific PR by number (e.g., `cursor-tools github pr 123`)
`cursor-tools github issue [number]` - Get the last 10 issues, or a specific issue by number (e.g., `cursor-tools github issue 456`)

**Browser Automation (Stateless):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oaknational) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
