---
trigger: always_on
description: - Always write clean, readable, and maintainable code
---

# Frontend Cursor AI Rules - Next.js + shadcn/ui + TypeScript

## General Development Rules

### Code Quality & Style
- Always write clean, readable, and maintainable code
- Use consistent naming conventions (camelCase for variables/functions, PascalCase for components)
- Add meaningful comments for complex logic
- Follow DRY (Don't Repeat Yourself) principles
- Implement proper error handling and logging
- Write self-documenting code with descriptive names

### TypeScript Standards
- Use strict TypeScript configuration
- Define proper interfaces and types for all data structures
- Avoid using 'any' type - use 'unknown' or proper typing instead
- Create custom types for API responses and component props
- Use proper generic types where applicable
- Implement proper type guards for runtime type checking

## Next.js Best Practices

### App Router Implementation
- Use App Router over Pages Router for new projects
- Implement proper Server Components vs Client Components separation
- Use Next.js built-in Image component for optimization
- Implement proper metadata and SEO optimization with next-intl locale support
- Use dynamic imports for code splitting
- Implement proper loading.tsx and error.tsx files with internationalization
- Use Server Actions for form submissions and mutations
- Implement proper caching strategies with revalidation
- Configure proper middleware for locale detection and routing
- Use generateStaticParams for static generation with multiple locales

### Server vs Client Components
```typescript
// Server Component (default)
export default async function ServerPage() {
  const data = await fetch('api/data')
  return <div>{data}</div>
}

// Client Component (when needed)
'use client'
import { useState } from 'react'

export default function ClientComponent() {
  const [state, setState] = useState('')
  return <input onChange={(e) => setState(e.target.value)} />
}
```

## shadcn/ui Component Usage

### Component Implementation
- Use shadcn/ui components as base building blocks
- Customize components through className props and CSS variables
- Create compound components by combining shadcn/ui primitives
- Use proper form components with react-hook-form integration
- Implement consistent theming through CSS variables
- Use Radix UI primitives understanding for advanced customization
- Follow shadcn/ui naming conventions for custom components

### Form Pattern with shadcn/ui
```typescript
const form = useForm<FormData>({
  resolver: zodResolver(formSchema),
  defaultValues: {...}
})

return (
  <Form {...form}>
    <FormField
      control={form.control}
      name="email"
      render={({ field }) => (
        <FormItem>
          <FormLabel>{t('email')}</FormLabel>
          <FormControl>
            <Input {...field} />
          </FormControl>
          <FormMessage />
        </FormItem>
      )}
    />
  </Form>
)
```

## React/Next.js Component Development

### Component Architecture
- Use functional components with hooks exclusively
- Implement proper state management with useState/useReducer
- Use Server Components by default, Client Components when needed
- Create reusable custom hooks for shared logic
- Implement proper prop validation with TypeScript interfaces
- Use React.memo() for expensive components
- Follow compound component patterns for complex UI

### Custom Hooks Pattern
```typescript
// Custom hook for API data
const useApiData = <T>(endpoint: string) => {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    // Fetch data logic
  }, [endpoint])

  return { data, loading, error }
}
```

## Styling and UI

### Tailwind CSS Implementation
- Use Tailwind CSS utility classes following shadcn/ui patterns
- Implement responsive design with Tailwind breakpoints
- Use CSS variables for theming (light/dark mode)
- Follow mobile-first responsive design principles
- Use proper spacing scale (4, 8, 16, 24, 32px multiples)
- Implement proper focus states and accessibility
- Use shadcn/ui theme system for consistent colors

### Theme Configuration
```typescript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        // ... other shadcn/ui colors
      }
    }
  }
}
```

## Animation Rules

### Framer Motion Best Practices
- Use Framer Motion for React component animations and transitions
- Implement layout animations with `layout` prop for smooth element repositioning
- Use `AnimatePresence` for enter/exit animations and route transitions
- Create reusable motion variants for consistent animation patterns
- Use `motion.div` and other motion components for declarative animations
- Implement proper `initial`, `animate`, and `exit` states
- Use `whileHover`, `whileTap`, and `whileFocus` for interactive animations
- Optimize performance with `layoutId` for shared element transitions
- Use `useAnimation` hook for imperative animation control
- Implement proper `transition` configurations for timing and easing

### Framer Motion Patterns
```typescript
// Page transitions
const pageVariants = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asbilim/modern-django-frontend](https://github.com/asbilim/modern-django-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
