---
trigger: always_on
description: This document describes the coding conventions and patterns used throughout this project.
---

# Code Conventions

This document describes the coding conventions and patterns used throughout this project.

## File Naming

| Type       | Convention       | Example                              |
| ---------- | ---------------- | ------------------------------------ |
| Components | `kebab-case.tsx` | `media-poster.tsx`                   |
| Pages      | `page.tsx`       | `app/(lists)/movie/popular/page.tsx` |
| Layouts    | `layout.tsx`     | `app/(detail)/movie/[id]/layout.tsx` |
| Loading    | `loading.tsx`    | `app/(lists)/loading.tsx`            |
| Hooks      | `camelCase.ts`   | `useFilters.ts`                      |
| Utils      | `kebab-case.ts`  | `lib/utils.ts`                       |
| Types      | `kebab-case.ts`  | `tmdb/models/movie.ts`               |
| Config     | `kebab-case.ts`  | `config/site.ts`                     |

## Naming Conventions

### Components

```tsx
// Filename: media-poster.tsx
// Component: PascalCase
export const MediaPoster: React.FC<MediaPosterProps> = () => { ... }

// Compound components
export const MediaCard = { Root, Content, Title, Excerpt }
```

### Functions & Variables

```tsx
// Functions: camelCase
function formatRuntime(minutes: number) { ... }

// Constants: camelCase or UPPER_SNAKE_CASE for true constants
const siteConfig = { ... }
const API_BASE_URL = "https://api.themoviedb.org/3"

// Boolean variables: is/has/should prefix
const isLoading = true
const hasCredits = credits.length > 0
```

### Types

```tsx
// Types: PascalCase
type Movie = { ... }
type MovieListType = "popular" | "top_rated" | "upcoming"

// Props interfaces: ComponentNameProps
interface MediaPosterProps { ... }

// Request params: DescriptiveRequestParams
type MovieListRequestParams = { ... }
```

## Export Patterns

### Named Exports (Required)

```tsx
// ✅ Correct: Named export
export const MovieCard: React.FC<Movie> = () => { ... }

// ❌ Wrong: Default export
export default function MovieCard() { ... }
```

### Barrel Exports

Re-export from index files for cleaner imports:

```tsx
// Usage
import { useFilters, usePagination } from "@/hooks"

// hooks/index.ts
export * from "./useFilters"
export * from "./usePagination"
export * from "./useSearch"
```

## Component Patterns

### Basic Component

```tsx
import { ComponentProps } from "react"

import { cn } from "@/lib/utils"

interface MediaPosterProps extends ComponentProps<"div"> {
  image?: string
  alt: string
  priority?: boolean
}

export const MediaPoster: React.FC<MediaPosterProps> = ({
  image,
  alt,
  priority,
  className,
  ...props
}) => {
  return (
    <div className={cn("relative aspect-poster", className)} {...props}>
      <Image
        src={image}
        alt={alt}
        priority={priority}
        fill
        className="object-cover"
      />
    </div>
  )
}
```

### Compound Component

```tsx
import { ComponentProps } from "react"

import { cn } from "@/lib/utils"

interface RootProps extends ComponentProps<"div"> {
  children: React.ReactNode
}

const Root: React.FC<RootProps> = ({ children, className, ...props }) => (
  <div
    className={cn("group relative overflow-hidden rounded-md", className)}
    {...props}
  >
    {children}
  </div>
)

interface TitleProps {
  children: React.ReactNode
}

const Title: React.FC<TitleProps> = ({ children }) => (
  <h3 className="truncate font-medium">{children}</h3>
)

const Content: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="space-y-1 p-2">{children}</div>
)

const Excerpt: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <p className="truncate text-xs text-muted-foreground">{children}</p>
)

export const MediaCard = { Root, Title, Content, Excerpt }
```

### Client Component

```tsx
"use client"

import { useState } from "react"
import { useSearchParams } from "next/navigation"

import { Button } from "@/components/ui/button"

export const SearchInput: React.FC = () => {
  const searchParams = useSearchParams()
  const [query, setQuery] = useState(searchParams.get("q") ?? "")

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />
}
```

## Props Patterns

### Extending HTML Elements

```tsx
// Extend native element props
interface ButtonProps extends ComponentProps<"button"> {
  variant?: "default" | "outline"
}

// For Radix components
interface DialogProps extends ComponentProps<typeof DialogPrimitive.Root> {
  title: string
}
```

### Using Domain Types

```tsx
// When component maps 1:1 with API data
import { Movie } from "@/tmdb/models"

export const MovieCard: React.FC<Movie> = ({
  id,
  title,
  poster_path,
  vote_average,
}) => { ... }
```

### Optional Props with Defaults

```tsx
interface MediaPosterProps {
  image?: string
  size?: "w300" | "w500" | "original"
  alt: string
}

export const MediaPoster: React.FC<MediaPosterProps> = ({
  image,
  size = "w500",  // Default value
  alt,
}) => { ... }
```

## Styling Patterns

### Class Merging

Always use `cn()` for conditional and merged classes:

```tsx
import { cn } from "@/lib/utils"

// Merge with className prop
<div className={cn("flex items-center gap-2", className)}>

// Conditional classes
<div className={cn(
  "rounded-md border",
  isActive && "border-primary",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oktay/movies](https://github.com/oktay/movies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
