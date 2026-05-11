---
trigger: always_on
description: This file is the single source of truth for repository-specific agent instructions.
---

# AGENTS.md

This file is the single source of truth for repository-specific agent instructions.
`CLAUDE.md` must only point here. If any other instruction file disagrees with this file, follow `AGENTS.md`.

## 1. Working Agreement

- Make minimal, correct, production-ready changes. Avoid over-engineering.
- Prefer simple modules, low cyclomatic complexity, and reusable logic.
- Check existing patterns before adding new abstractions.
- Keep user-visible behavior complete. Do not leave mock data, placeholder flows, or half-finished paths.
- When documentation in this file conflicts with the codebase, verify the codebase and update the documentation to match reality.

## 2. Development Commands

### Core commands

```bash
pnpm install
pnpm dev
pnpm build
pnpm start
pnpm lint
pnpm type-check
pnpm test
pnpm test:watch
pnpm test:coverage
pnpm analyze
pnpm analyze:dev
pnpm prettier:check
pnpm prettier:format
```

### Database commands

```bash
pnpm db:generate
pnpm db:migrate
pnpm db:push
```

### Utilities

```bash
pnpm set:admin
```

## 3. Project Snapshot

- Framework: Next.js 16 App Router
- Runtime UI stack: React 19, Tailwind CSS v4, shadcn/ui, Radix UI
- Package manager: `pnpm`
- Database: PostgreSQL with Drizzle ORM
- Auth: Better Auth, magic link via Resend, optional OAuth providers
- Billing: provider abstraction in `src/lib/billing/provider.ts`, current implementation uses Creem
- Storage: Cloudflare R2 with presigned uploads
- Content: Content Collections plus repository-managed Markdown
- Localization: `@lingo.dev/compiler`

## 4. Important Paths

- App routes: `src/app`
- Public marketing pages: `src/app/(pages)`
- Auth routes: `src/app/(auth)`
- Protected app: `src/app/dashboard`
- API routes: `src/app/api`
- Shared components: `src/components`
- UI primitives: `src/components/ui`
- Forms: `src/components/forms`
- Business logic: `src/lib`
- Auth logic: `src/lib/auth`
- Billing logic: `src/lib/billing`
- i18n helpers: `src/lib/i18n`, `src/lib/config/i18n.ts`, `src/lib/config/i18n-routing.ts`
- Database schema and migrations: `src/database`, `src/database/migrations`
- Email templates: `src/emails`
- Environment validation: `env.js`
- Route protection: `src/proxy.ts`
- Next config: `next.config.ts`

## 5. Engineering Rules

### TypeScript and module design

- Keep types explicit and strict. Do not introduce `any`.
- Prefer small focused modules over large files.
- Reuse existing components and utilities before creating new ones.
- Follow repository naming patterns:
  - Components: PascalCase
  - Functions and variables: camelCase
- Keep code readable without clever indirection. If a pattern needs a long explanation, it is probably too complex.

### Next.js and React

- Default to Server Components. Add Client Components only when interactivity or browser APIs require them.
- Push `"use client"` down to the smallest interactive leaf.
- Follow App Router conventions for pages, layouts, route handlers, and colocated `_components`.
- Use real implementations. Do not ship fake data, fake success states, or dead-end UI.
- Add page metadata where appropriate.

### Layout and page width

- Use the semantic containers from `src/components/layout/page-container.tsx` instead of page-local `max-w-*` wrappers when working outside the dashboard.
- Use `ShellContainer` for global chrome and genuinely wide split layouts such as the marketing header, footer, and homepage hero.
- Use `SectionContainer` for standard marketing sections and most non-dashboard page bodies.
- Use `ReadingContainer` for article content, legal copy, and other long-form reading surfaces.
- Use `CompactContainer` for narrow auth flows.
- Use `FocusContainer` for status pages and single-card flows that need more space.
- Treat full-bleed backgrounds and content width as separate concerns: a section may span the viewport, but its content should still sit inside one semantic container.
- Do not add new ad hoc width systems or scatter `max-w-*` utilities through page modules unless a one-off component truly cannot be expressed with the existing containers.

### Forms and validation

- Use React Hook Form with Zod for form validation.
- Keep schemas close to the form or in a clearly named schema module.
- Handle loading, validation, and error states explicitly.

## 6. Localization and Lingo

### Core policy

- Do not hardcode user-visible language in code. Use the repository's Lingo-based i18n flow.
- Treat free-form business data such as `banReason` as raw content. It does not need translation, but any surrounding UI copy still must follow i18n rules.
- Source copy should be authored in English unless a feature explicitly requires a different canonical source language.
- Shipped UI must respect the active locale and must not mix English with translated copy in the same view.

### Source of truth for locale behavior

- Keep locale detection and persistence in `src/.lingo/locale-resolver.server.ts` and `src/.lingo/locale-resolver.client.ts`.
- In server code, read request locale through `src/lib/i18n/server-locale.ts`.
- For client-side locale switching, prefer `useLingoContext().setLocale()` for same-route updates.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UllrAI/SaaS-Starter](https://github.com/UllrAI/SaaS-Starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
