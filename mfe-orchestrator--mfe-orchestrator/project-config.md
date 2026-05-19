---
trigger: always_on
description: This is a **monorepo** microfrontend orchestrator project with:
---

# Microfrontend Orchestrator - Cursor Rules

## Project Overview

This is a **monorepo** microfrontend orchestrator project with:

- **📦 Monorepo**: pnpm workspace with centralized dependency management
- **⚡ Turbo**: Build system for optimized task orchestration and caching
- **🎨 Biome**: Unified linting and formatting across all packages
- **🪝 Lefthook**: Git hooks for automated code quality checks
- **📋 Commitlint**: Enforced conventional commit messages

### Architecture

- **Backend**: Fastify (Node.js/TypeScript) with MongoDB, Redis
- **Frontend**: React with TypeScript, React Router, TanStack Query, Zustand
- Uses shadcn/ui components, Tailwind CSS, i18n for internationalization

### Workspace Structure

```
/
├── backend/           # Fastify backend package
├── frontend/          # React frontend package
├── package.json       # Workspace root with shared scripts
├── pnpm-workspace.yaml # Workspace configuration
├── turbo.json         # Turbo build configuration
├── biome.json         # Biome configuration for all packages
├── lefthook.yml       # Git hooks configuration
└── commitlint.config.js # Commit message validation
```

---

## Frontend Rules

### Routing & Navigation

- ❌ **NEVER use Next.js router** - We use React Router with `react-router-dom`
- All pages must be located under `src/pages/`
- All components must be located under `src/components/`
- When creating a new page:
  1. Add the route in `src/Routes.tsx`
  2. Add the navigation link in the sidebar (`src/theme/layout/MainLayout.tsx` or related sidebar components)
- Use lazy loading for all page components
- Use `<Suspense>` with `<Spinner />` for route loading states

### Forms & Validation

- Always use **react-hook-form** for forms
- Use form components from `src/components/ui/form`
- Use input components from `src/components/input/` (TextField.rhf, SelectField.rhf, etc.)
- Don't use `onError` callback in `useQuery` hooks

### Internationalization

- Internationalize everything using the **i18n module**
- Use translations from `public/locales/{lang}/` folders
- Existing languages: `en`, `it`
- Use the `useTranslation` hook from `react-i18next`

### UI Components

- Always reuse existing **UI elements** from `src/components/ui/`
- Use components from shadcn/ui library (`@/components/ui/`)
- Use icons from **lucide-react** only
- Don't create new UI components from scratch - check existing ones first

### API Calls & Data Fetching

- For API calls on page load, use `import { useQuery } from '@tanstack/react-query'`
- Wrap the component with `<ApiStatusHandler queries={[dataQuery]}>`
- Example:

```tsx
const dataQuery = useQuery({
  queryKey: ["key", param],
  queryFn: () => apiFunction(param),
});
return (
  <ApiStatusHandler queries={[dataQuery]}>
    {/* Your component content */}
  </ApiStatusHandler>
);
```

- Never use `onError` in `useQuery` hooks
- The API client/hooks are located under `src/hooks/` or `src/api/`

### Toast Notifications

- Use Zustand store for toast notifications
- Import: `import useToastNotificationStore from '@/store/useToastNotificationStore';`
- Usage:

```tsx
const notifications = useToastNotificationStore();
notifications.showSuccessNotification({ message: "message" });
notifications.showErrorNotification({ message: "message" });
notifications.showWarningNotification({ message: "message" });
notifications.showInfoNotification({ message: "message" });
```

### Component Structure

- All pages in `src/pages/`
- All components in `src/components/`
- Use TypeScript for all components
- Use functional components with TypeScript interfaces

### State Management

- Use Zustand for global state (see `src/store/`)
- Use React Query for server state caching
- Use local state for component-specific state

---

## Backend Rules

### Architecture Pattern

The backend follows a layered architecture:

- **Models**: `src/models/` - Mongoose schemas and interfaces
- **Services**: `src/service/` - Business logic layer
- **Controllers**: `src/controller/` - API route handlers
- **Plugins**: `src/plugins/` - Fastify plugins (autoloaded)

### Controller Pattern

- Controllers are auto-loaded from `src/controller/` directory
- Export as default async function: `export default async function controllerName(fastify: FastifyInstance)`
- Access database user from `request.databaseUser`
- Use service layer for business logic
- Example structure:

```typescript
export default async function myController(fastify: FastifyInstance) {
  fastify.get("/endpoint", async (request, reply) => {
    const result = await new MyService(request.databaseUser).method();
    return reply.send(result);
  });
}
```

### Service Pattern

- Services extend `BaseAuthorizedService` for authorization checks
- Services receive `databaseUser` in constructor
- Services contain business logic, data validation, and database operations
- Example:

```typescript
export class MyService extends BaseAuthorizedService {
  constructor(user?: IUser) {
    super(user);
  }

  async method(param: string) {
    // Business logic here
    return result;
  }
}
```

### Model Pattern

- Models use Mongoose schemas
- Define TypeScript interface extending `Document<ObjectId>`
- Use timestamps: `{ timestamps: true }`
- Export both model and interface
- Example:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mfe-orchestrator/mfe-orchestrator](https://github.com/mfe-orchestrator/mfe-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
