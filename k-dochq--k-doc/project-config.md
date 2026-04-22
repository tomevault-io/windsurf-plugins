---
trigger: always_on
description: This rule explains Next.js conventions and best practices for fullstack development.
---


# Next.js rules

- Use the App Router structure with `page.tsx` files in route directories.
- Client components must be explicitly marked with `'use client'` at the top of the file.
- Use kebab-case for directory names (e.g., `components/auth-form`) and PascalCase for component
  files.
- Prefer named exports over default exports, i.e. `export function Button() { /* ... */ }` instead
  of `export default function Button() { /* ... */ }`.
- Minimize `'use client'` directives:
  - Keep most components as React Server Components (RSC)
  - Only use client components when you need interactivity and wrap in `Suspense` with fallback UI
  - Create small client component wrappers around interactive elements
- Avoid unnecessary `useState` and `useEffect` when possible:
  - Use server components for data fetching
  - Use URL search params for shareable state
- Use `nuqs` for URL search param state management

## Internationalization (i18n) Rules

### Locale-Aware Navigation

- **Never use raw `useRouter` or `Link`** for internal navigation in internationalized apps
- **Always use `LocaleLink` component** for internal links that need locale prefixing
- **Always use `useLocalizedRouter` hook** instead of raw `useRouter` for programmatic navigation

#### LocaleLink Usage

```typescript
// ✅ Good - Automatic locale handling
import { LocaleLink } from 'shared/ui/locale-link';

<LocaleLink href="/auth/signup">
  회원가입
</LocaleLink>

// ✅ With custom locale
<LocaleLink href="/auth/signup" locale="en">
  Sign Up
</LocaleLink>
```

#### useLocalizedRouter Usage

```typescript
// ✅ Good - Automatic locale prefixing
import { useLocalizedRouter } from 'shared/model/hooks/useLocalizedRouter';

function MyComponent() {
  const router = useLocalizedRouter();

  const handleNavigate = () => {
    router.push('/auth/signup'); // Automatically becomes /en/auth/signup
    router.replace('/dashboard'); // Automatically becomes /en/dashboard
  };

  return <button onClick={handleNavigate}>Navigate</button>;
}

// ❌ Bad - Manual locale handling
import { useRouter, usePathname } from 'next/navigation';
import { DEFAULT_LOCALE, type Locale } from 'shared/config';

function MyComponent() {
  const router = useRouter();
  const pathname = usePathname();

  const currentLocale = (pathname.split('/')[1] as Locale) || DEFAULT_LOCALE;

  const handleNavigate = () => {
    router.push(`/${currentLocale}/auth/signup`); // Manual locale handling
  };
}
```

### Benefits of Locale-Aware Navigation

- **Consistency**: All internal links automatically include correct locale
- **Maintainability**: Locale logic centralized in reusable components
- **Developer Experience**: No need to manually handle locale in every component
- **Error Prevention**: Prevents broken links due to missing locale prefixes

## TypeScript Rules

- **Never use `any` type**: Always provide specific types for better type safety
  - Use `unknown` for truly unknown types that need type guards
  - Use union types like `string | number` for multiple possible types
  - Use generics `<T>` for reusable type-safe functions
  - Use proper interface or type definitions instead of `any`

### Page Props Type Safety

- **Always use Promise types for page props** in Next.js App Router (Next.js 15)
- **`params` and `searchParams` are now Promise types** and must be awaited

#### ✅ Correct Page Props Usage

```typescript
interface PageProps {
  params: Promise<{
    lang: Locale;
    id?: string;
  }>;
  searchParams: Promise<{
    error?: string;
    provider?: string;
  }>;
}

export default async function Page({ params, searchParams }: PageProps) {
  const { lang, id } = await params;
  const resolvedSearchParams = await searchParams;

  // Use lang, id, resolvedSearchParams.error, etc.
}
```

#### ❌ Incorrect Page Props Usage

```typescript
interface PageProps {
  params: {
    lang: Locale;
  };
  searchParams: {
    error?: string;
  };
}

export default async function Page({ params, searchParams }: PageProps) {
  // ❌ This will cause TypeScript errors
  const dict = await getDictionary(params.lang);
  const error = searchParams.error;
}
```

#### Common Build Errors to Avoid

- **Type error**: `Type 'PageProps' does not satisfy the constraint 'PageProps'`
- **Missing properties**: `Types of property 'params' are incompatible`
- **Promise methods**: `Type is missing the following properties: then, catch, finally`

#### Migration Pattern

```typescript
// Before (Next.js 13)
interface Props {
  params: { lang: string };
  searchParams: { error?: string };
}

// After (Next.js 15)
interface Props {
  params: Promise<{ lang: string }>;
  searchParams: Promise<{ error?: string }>;
}

export default async function Page({ params, searchParams }: Props) {
  const { lang } = await params;
  const { error } = await searchParams;
  // ...
}
```

## TanStack Query Best Practices

### Setup and Configuration

- Create a QueryClient wrapper with proper configuration:

```typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

function makeQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000, // 1 minute
        gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/k-dochq) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
