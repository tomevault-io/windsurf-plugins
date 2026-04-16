---
trigger: always_on
description: > **⚠️ Maintenance Note for Future Agents**: This file contains critical patterns and conventions discovered through codebase exploration. When you add new important patterns, utilities, or architectural decisions, please update this file to help future context gathering. Focus on patterns that would save time if documented upfront (e.g., utility functions, common code patterns, configuration details).
---

# Daemon Project Rules

> **⚠️ Maintenance Note for Future Agents**: This file contains critical patterns and conventions discovered through codebase exploration. When you add new important patterns, utilities, or architectural decisions, please update this file to help future context gathering. Focus on patterns that would save time if documented upfront (e.g., utility functions, common code patterns, configuration details).

## Project Context

You are working on Daemon, an AI agent deployment platform. The project uses:

- **Frontend**: Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS
- **Backend**: Supabase (PostgreSQL, Auth, RLS)
- **Authentication**: OAuth-only (Google, GitHub)
- **Monorepo**: Turborepo with apps/web, packages/cli, packages/db
- **Styling**: Tailwind + shadcn/ui components
- **Database**: PostgreSQL with Row Level Security (RLS)

## Code Style & Conventions

### TypeScript

- Use TypeScript for all code
- Prefer `async/await` over promises
- Use type inference where possible
- Avoid `any` types - use `unknown` if needed

### React & Next.js

- Use Server Components by default
- Only use "use client" when necessary (state, effects, browser APIs)
- Prefer server-side data fetching with `async` functions
- Use Next.js App Router conventions (not Pages Router)

### Code Organization

- Keep files small and focused (under 300 lines)
- Return early to avoid else clauses
- Prefer composition over inheritance
- Use named exports over default exports

### Naming Conventions

- Files: `kebab-case.tsx`
- Components: `PascalCase`
- Functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Database tables: `snake_case`

## Supabase & Database

### Authentication

- OAuth-only (Google, GitHub) - no email/password
- Use `@/lib/supabase/server` for Server Components
- Use `@/lib/supabase/client` for Client Components
- Always check auth before protected operations
- User profiles auto-created via database trigger

### Database Access

- All tables have Row Level Security (RLS) enabled
- Use team-based access control via `team_members` table
- Never bypass RLS in application code
- Service role key only for admin operations

### Queries

- Prefer server-side queries with proper RLS
- Use `getUser()` helper for auth checks
- Handle errors gracefully
- Log database errors for debugging

### Supabase Client Patterns

- **Server Components**: Use `createClient()` from `@/lib/supabase/server` (auth-aware, uses cookies)
- **Client Components**: Use `createClient()` from `@/lib/supabase/client` (browser client)
- **Admin Operations**: Use `createSupabaseServer()` from `@/lib/supabase` (service role, bypasses RLS)
- **Middleware**: Session refresh handled in `@/lib/supabase/middleware.ts` - call `updateSession()` in Next.js middleware
- **Helpers**: `getUser()`, `getUserProfile()`, `getSession()` available from `@/lib/supabase/server`

## UI & Styling

### Design System

- Use shadcn/ui components from `@/components/ui`
- Follow existing component patterns
- Dark mode first, light mode compatible
- Use Tailwind utility classes, avoid custom CSS

### Tailwind CSS Gotchas

- **Opacity modifiers with CSS variables**: To robustly support opacity modifiers (e.g., `border-border/80`) with CSS variables, use a helper function in `tailwind.config.ts` instead of string placeholders. This avoids parsing ambiguities.
  ```ts
  function withOpacity(variableName: string) {
    return ({ opacityValue }: { opacityValue?: string }) => {
      if (opacityValue !== undefined) {
        return `hsl(var(${variableName}), ${opacityValue})`;
      }
      return `hsl(var(${variableName}))`;
    };
  }
  // usage: colors: { border: withOpacity("--border") }
  ```

### Components

- Keep components simple and reusable
- Use TypeScript interfaces for props
- Handle loading and error states
- Optimize for performance (use `memo` sparingly)

### UI Patterns

- **No slide-out panels (Sheet)** - Use pages or modals (Dialog) instead
- Use `Dialog` from shadcn/ui for quick actions, confirmations, or forms that don't need a full page
- Use dedicated pages for complex content that benefits from URL routing
- Mobile navigation should use full-screen pages, not slide-outs

### Accessibility

- Use semantic HTML
- Include proper ARIA labels
- Ensure keyboard navigation works
- Test with screen readers

## Error Handling

### Frontend

- Show user-friendly error messages
- Log errors to console for debugging
- Handle loading states properly
- Provide fallback UI when needed

### Backend

- Return appropriate HTTP status codes
- Include helpful error messages
- Never expose sensitive data in errors
- Log errors with context

## Security

### Authentication & Authorization

- Always verify user authentication server-side
- Check team membership for multi-tenant features
- Use RLS policies for database access
- Validate all user input

### Data Protection


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daemon-so) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
