---
trigger: always_on
description: This document provides instructions and best practices for GitHub Copilot when working with this Next.js project.
---

# GitHub Copilot Instructions for Next.js Project

This document provides instructions and best practices for GitHub Copilot when working with this Next.js project.

## Project Structure

- Follow the App Router file structure convention:
  - `app/` directory for routes and layouts
  - `components/` for reusable UI components
  - `lib/` for utility functions and shared code
  - `public/` for static assets
  - `styles/` for global CSS and styling

## Color Theme

This job matching with CV platform uses the following professional color theme:

- **Primary**: `#2563EB` (Royal Blue) - Used for primary buttons, main navigation elements, and branded areas
- **Secondary**: `#475569` (Slate Gray) - Used for secondary UI elements, subheadings, and supporting content
- **Accent**: `#10B981` (Emerald Green) - Used for accents, success indicators, calls-to-action, and highlighting matches
- **Neutral Background**: `#F8FAFC` (Light Gray) - Used for page backgrounds and cards
- **Text Primary**: `#1E293B` (Dark Slate) - Main text color
- **Text Secondary**: `#64748B` (Medium Gray) - Secondary text and less important information

This color scheme conveys professionalism, trust, and success - perfect for a job matching platform. The blue represents reliability and professionalism, while the green accent indicates success and growth, appropriate for highlighting matches and positive outcomes.

## Code Style & Best Practices

### General

- Use TypeScript for all new files
- Prefer functional components with hooks over class components
- Use named exports instead of default exports where appropriate
- Keep components small and focused on a single responsibility
- Write self-documenting code with descriptive variable/function names
- Use absolute imports instead of relative path hell (e.g., `@/components/Button` instead of `../../../components/Button`)
- Use shadcn for the UI components

### React & Next.js

- Use Server Components by default unless client interactivity is needed
- Add "use client" directive only to components that:
  - Use React hooks
  - Need browser APIs
  - Require event listeners
  - Manage client-side state
- Use the `Link` component for internal navigation, not `<a>` tags
- Implement proper loading states and error boundaries
- Use Next.js Image component for optimized images
- Implement proper metadata for SEO using metadata objects or generateMetadata

### Data Fetching

- Prefer server-side data fetching in Server Components
- Use React Query for client-side data fetching when needed
- Implement proper loading, error, and empty states
- Use revalidation strategies appropriately (ISR with revalidate, on-demand revalidation)

### State Management

- Use React Context API for global state when needed
- Prefer local component state with useState/useReducer for component-specific state
- Consider Zustand for more complex state management needs

### Styling

- Use CSS Modules or Tailwind CSS for component styling
- Follow mobile-first responsive design approach
- Maintain a consistent design system with reusable variables/themes
- Implement the color theme specified above throughout the application
- Use the color theme variables in a Tailwind config or CSS variables

### Forms

- Use controlled components for form inputs when appropriate
- Implement proper form validation with helpful error messages
- Use React Hook Form for complex forms

### Performance

- Implement code splitting using dynamic imports where beneficial
- Optimize and lazy-load images
- Use Next.js font optimization
- Memoize expensive computations with useMemo and useCallback when appropriate
- Virtualize long lists with react-window or similar libraries

### Accessibility

- Use semantic HTML elements
- Ensure proper keyboard navigation
- Add appropriate ARIA attributes when necessary
- Maintain sufficient color contrast with the chosen color theme
- Support screen readers with proper labels and descriptions

### Testing

- Write unit tests for utility functions
- Write component tests for UI components
- Add end-to-end tests for critical user flows
- Implement proper test coverage

### Error Handling

- Use try/catch blocks for error handling
- Create custom error handling components
- Implement proper error logging

## Common Patterns to Follow

### API Routes

```typescript
// Example API route pattern with proper error handling
export async function GET(request: Request) {
  try {
    // Logic here
    return Response.json({ data: result })
  } catch (error) {
    console.error('Failed to process request:', error)
    return Response.json(
      { error: 'Something went wrong' },
      { status: 500 }
    )
  }
}
```

### Server Component

```typescript
// Server component pattern
import { getItems } from '@/lib/data'

export default async function ItemsList() {
  // Data fetching directly in server component
  const items = await getItems()
  
  return (
    <ul className="grid gap-4">
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}
```

### Client Component

```typescript
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <div>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makokhavictor/job-matcher](https://github.com/makokhavictor/job-matcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
