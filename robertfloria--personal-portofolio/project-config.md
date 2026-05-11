---
trigger: always_on
description: This is a **personal portfolio monorepo** built with modern web technologies. The application showcases professional experience, projects, skills, and provides contact functionality with CV download capabilities.
---

# Copilot Instructions - Personal Portfolio Monorepo

## Project Overview

This is a **personal portfolio monorepo** built with modern web technologies. The application showcases professional experience, projects, skills, and provides contact functionality with CV download capabilities.

## Architecture

### Monorepo Structure (npm Workspaces)

```
personal-portofolio/
├── apps/
│   ├── web/          # Next.js 16 frontend (React 19)
│   └── api/          # NestJS backend
├── packages/
│   ├── shared-types/ # Shared TypeScript interfaces/types
│   └── shared-utils/ # Shared utility functions
└── package.json      # Root workspace configuration
```

### Technology Stack

| Layer                | Technology                                       |
| -------------------- | ------------------------------------------------ |
| **Frontend**         | Next.js 16, React 19, TypeScript, Tailwind CSS 4 |
| **Backend**          | NestJS 11, TypeScript                            |
| **State Management** | React Query (TanStack Query), React Context      |
| **Forms**            | React Hook Form + Zod validation                 |
| **Styling**          | Tailwind CSS, class-variance-authority (CVA)     |
| **Animations**       | Framer Motion, Lottie                            |
| **HTTP Client**      | Axios (API), Fetch (Services)                    |
| **Testing**          | Jest, React Testing Library                      |
| **Icons**            | Lucide React                                     |

---

## Frontend Architecture (`apps/web`)

### Directory Structure

```
apps/web/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root layout with providers
│   ├── page.tsx            # Home page entry
│   ├── globals.css         # Global styles & CSS variables
│   └── api/                # Next.js API routes (proxy to NestJS)
│       ├── cv/route.ts     # CV download proxy
│       └── email/route.ts  # Email sending proxy
├── components/
│   ├── common/             # Reusable UI components
│   └── layout/             # Layout components (Navbar, Footer, Providers)
├── features/
│   └── home/               # Feature-based organization
│       ├── index.tsx       # Feature entry point
│       └── sections/       # Page sections
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities, constants, HTTP client
├── services/               # API service layer
├── store/
│   └── contexts/           # React Context providers
├── types/                  # TypeScript type definitions
└── public/                 # Static assets (images, lottie)
```

### Component Patterns

#### 1. Compound Component Pattern

Used for complex UI components like `Card` and `Section`:

```tsx
// Usage example
<Card.Root variant="glass" hover="glow" animated>
  <Card.Header>
    <Card.Title>Title</Card.Title>
    <Card.Description>Description</Card.Description>
  </Card.Header>
  <Card.Content>Content</Card.Content>
  <Card.Footer>Footer</Card.Footer>
</Card.Root>

<Section.Root id="about">
  <Section.Header title="About" subtitle="Subtitle" highlightText="Me" />
  <Section.Content>Content</Section.Content>
</Section.Root>
```

#### 2. CVA (Class Variance Authority) for Variants

All UI components use CVA for consistent variant styling:

```tsx
const buttonVariants = cva('base-classes', {
  variants: {
    variant: { primary: '...', secondary: '...', outline: '...', ghost: '...', link: '...' },
    size: { sm: '...', md: '...', lg: '...', icon: '...' },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});
```

#### 3. Feature-Based Organization

Each section follows this structure:

```
sections/hero-section/
├── index.tsx           # Main section component
├── index.test.tsx      # Tests
├── components/         # Section-specific components
│   ├── index.ts        # Re-exports
│   ├── hero-text.tsx
│   └── profile-image.tsx
└── lib/
    └── data.ts         # Section-specific data
```

### State Management

#### React Query for Server State

```tsx
// Mutations with automatic notifications
const mutation = useMutationWithNotification({
  mutationFn: async (data) => await service.action(data),
});
```

#### React Context for UI State

- **UIContext**: Theme, sidebar, mobile menu state
- **NotificationContext**: Global toast notifications

### Custom Hooks

| Hook                          | Purpose                                      |
| ----------------------------- | -------------------------------------------- |
| `useMutationWithNotification` | React Query mutation with auto notifications |
| `useQueryWithNotification`    | React Query query with auto notifications    |
| `useSendEmail`                | Email sending mutation                       |
| `useCvDownload`               | CV download functionality                    |
| `useIntersectionObserver`     | Viewport visibility detection                |
| `useLockBodyScroll`           | Scroll locking for modals                    |
| `useGroupBy`                  | Array grouping utility                       |

### Service Layer Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertfloria/personal-portofolio](https://github.com/robertfloria/personal-portofolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
