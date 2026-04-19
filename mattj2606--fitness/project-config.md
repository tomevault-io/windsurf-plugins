---
trigger: always_on
description: This project is a personal, self-hosted fitness intelligence app designed for daily use on mobile (iPhone) via a mobile-first web app. The app focuses on fast daily check-ins, structured workout logging, muscle-level analytics, and simple, explainable recommendations.
---

# Fitness App - Cursor Rules

This project is a personal, self-hosted fitness intelligence app designed for daily use on mobile (iPhone) via a mobile-first web app. The app focuses on fast daily check-ins, structured workout logging, muscle-level analytics, and simple, explainable recommendations.

## Project Overview

- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **Platform**: Mobile-first PWA
- **Pattern**: React Server Components + API Routes
- **Focus**: Data ingestion interface - optimized for fast, mobile data entry

**Key URLs**:
- GitHub Repository: (to be added)

## Project Structure

```
app/
  (routes)/          # Route groups for organization
  api/               # API routes (route.ts files)
  components/        # Reusable components
    ui/              # Base UI components
    checkin/         # Feature-specific components
    workout/
    analytics/
  lib/               # Utilities and shared code
    db/              # Database utilities (Prisma)
    services/        # Business logic
    utils/           # Helper functions
    types/           # TypeScript types
```

## Next.js App Router Best Practices

1. **Use Server Components by default** - Only use Client Components when necessary for interactivity
2. **Implement the file-based routing system** - Use `app/` directory structure
3. **Use `layout.tsx` for shared layouts** - Nest layouts for route groups
4. **Implement `loading.tsx` for loading states** - Automatic Suspense boundaries
5. **Use `error.tsx` for error handling** - Route-level error boundaries
6. **Utilize route handlers (`route.ts`) for API routes** - Not Pages Router API routes
7. **Use metadata API for SEO** - Export metadata from page/layout files
8. **Use Next.js Image component** - Optimized image loading with WebP support
9. **Use environment variables** - `.env.local` for configuration

## TypeScript Best Practices

- **Use TypeScript for all code** - Prefer interfaces over types
- **Avoid enums** - Use maps or const objects instead
- **Use functional components with TypeScript interfaces**
- **Avoid `any`** - Use `unknown` if needed with proper type guards
- **Use Prisma-generated types** - Leverage type safety from database schema
- **Use Zod for runtime validation** - Form validation and API boundary validation

## Component Patterns

### Component Definition Syntax

```tsx
// Server Component (default)
export const ComponentName = () => {
  // Component logic
};

// Client Component (when needed)
'use client'

interface ComponentNameProps {
  // Props definition
}

export const ComponentName = ({ prop1, prop2 }: ComponentNameProps) => {
  // Component logic
};

// Page components use default exports
const Page = () => {
  // Page component logic
};

export default Page;
```

### File Organization Structure

For component files, structure in this order:
1. Exported component
2. Subcomponents
3. Helper functions
4. Static content
5. Types/interfaces

### Naming Conventions

- **Components**: PascalCase (`CheckinForm.tsx`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Constants**: UPPER_SNAKE_CASE (`API_ENDPOINTS.ts`)
- **Types/Interfaces**: PascalCase (`WorkoutData.ts`)
- **Directories**: lowercase with dashes (`components/auth-wizard/`)
- **API routes**: `route.ts` (lowercase)
- **Favor named exports** for components

## Code Style and Structure

- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`)
- Use the "function" keyword for pure functions
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements
- Use declarative JSX
- Minimize AI-generated comments; use clearly named variables and functions instead

## Error Handling and Validation

- **Prioritize error handling**: Handle errors and edge cases early
- **Use early returns and guard clauses**
- **Implement proper error logging** with user-friendly messages
- **Use Zod for form validation** - Type-safe validation schemas
- **Model expected errors as return values** in Server Actions
- **Use error boundaries** (`error.tsx`) for unexpected errors
- **Never expose sensitive error details** to client
- **Use consistent error response format** in API routes

## UI and Styling

- **Use Tailwind CSS exclusively** for styling - Avoid inline styles
- **Mobile-first approach**: Base styles for mobile, then `md:`, `lg:` breakpoints
- **Ensure touch targets are at least 44x44px** for mobile
- **Use consistent spacing scale** from Tailwind
- **Test on mobile viewport sizes** regularly
- **Implement responsive design** with Tailwind CSS breakpoints
- **Use semantic HTML** for accessibility
- **Include ARIA labels** where needed
- **Ensure keyboard navigation** works
- **Maintain proper heading hierarchy**
- **Provide alt text for images**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattj2606) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
