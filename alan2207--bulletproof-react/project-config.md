---
trigger: always_on
description: Bulletproof React is a scalable React application architecture that provides opinionated guidelines and best practices for building production-ready React applications. The project includes three different implementations:
---

# AGENTS.md

## Project Overview

Bulletproof React is a scalable React application architecture that provides opinionated guidelines and best practices for building production-ready React applications. The project includes three different implementations:

- **React Vite**: Modern Vite-based React application
- **Next.js App Router**: Next.js 13+ with App Router
- **Next.js Pages**: Traditional Next.js with Pages Router

### Application Domain
The demo application is a team collaboration platform where users can:
- Create and join teams
- Start discussions within teams  
- Comment on discussions
- Manage user roles (ADMIN/USER permissions)

**Live Demo**: [https://bulletproof-react-app.netlify.app](https://bulletproof-react-app.netlify.app)

## Setup Commands

```bash
# Navigate to desired app
cd apps/react-vite        # or apps/nextjs-app or apps/nextjs-pages

# Install dependencies
yarn install

# Start development server
yarn dev

# Run tests
yarn test

# Run e2e tests
yarn test:e2e

# Lint code
yarn lint

# Build for production
yarn build
```

## Project Structure

The codebase follows a feature-based architecture organized as follows:

```
src/
├── app/              # Application layer (routes, providers, router)
├── components/       # Shared UI components
├── config/          # Global configurations and env variables
├── features/        # Feature-based modules (auth, discussions, comments, etc.)
├── hooks/           # Shared React hooks
├── lib/             # Preconfigured libraries (react-query, auth, etc.)
├── testing/         # Test utilities and mocks
├── types/           # Shared TypeScript types
└── utils/           # Shared utility functions
```

### Feature Structure
Each feature should be self-contained:

```
src/features/awesome-feature/
├── api/         # API calls and hooks for this feature
├── components/  # Feature-specific components  
├── hooks/       # Feature-specific hooks
├── stores/      # Feature-specific state
├── types/       # Feature-specific types
└── utils/       # Feature-specific utilities
```

## Code Standards

### TypeScript
- **Strict mode enabled** - All TypeScript strict checks are enforced
- **Type-first approach** - Define types before implementation
- **Absolute imports** - Use `@/` prefix for all src imports (e.g., `@/components/ui/button`)

### Code Style
- **ESLint + Prettier** configured for consistent formatting
- **Kebab-case** for file and folder names
- **PascalCase** for React components
- **camelCase** for functions and variables

### Architecture Rules
- **No cross-feature imports** - Features should not import from each other
- **Unidirectional flow** - Code flows: shared → features → app
- **Colocation** - Keep related code as close as possible to where it's used

## Component Guidelines

### Best Practices
- **Composition over props** - Use children/slots instead of many props
- **Single responsibility** - Each component should have one clear purpose  
- **Extract render functions** - Move complex JSX into separate components
- **Limit prop count** - Consider composition if accepting too many props

### Styling
- **Tailwind CSS** is the primary styling solution
- **Headless UI components** using Radix UI primitives
- **ShadCN/UI pattern** - Components are copied into codebase, not installed as packages

## State Management Strategy

### Component State
- Use `useState` for simple independent state
- Use `useReducer` for complex state with multiple related updates

### Application State  
- **Zustand** for global application state (modals, notifications, themes)
- Keep state as close to usage as possible
- Avoid premature globalization

### Server State
- **React Query (TanStack Query)** for all server state management
- **MSW (Mock Service Worker)** for API mocking during development
- Separate fetcher functions from hooks

### Form State
- **React Hook Form** for form management
- **Zod** for form validation schemas
- Create reusable Form and Input components

## API Layer

### Structure
Each API endpoint should have:
1. **Types & validation schemas** for request/response
2. **Fetcher function** using configured API client
3. **React Query hook** for data fetching/caching

### Example Pattern
```typescript
// api/get-discussions.ts
export const getDiscussions = (params: GetDiscussionsParams): Promise<Discussion[]> => {
  return api.get('/discussions', { params });
};

export const useDiscussions = (params: GetDiscussionsParams) => {
  return useQuery({
    queryKey: ['discussions', params],
    queryFn: () => getDiscussions(params),
  });
};
```

## Testing Strategy

### Testing Pyramid
1. **Integration Tests** (primary focus) - Test feature workflows
2. **Unit Tests** - Test shared utilities and complex logic
3. **E2E Tests** - Test critical user journeys

### Tools
- **Vitest** - Test runner (Jest-compatible but faster)
- **Testing Library** - Component testing utilities
- **Playwright** - E2E testing framework
- **MSW** - API mocking for tests

### Testing Patterns
- Test behavior, not implementation details
- Use real HTTP requests with MSW instead of mocking fetch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alan2207/bulletproof-react](https://github.com/alan2207/bulletproof-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
