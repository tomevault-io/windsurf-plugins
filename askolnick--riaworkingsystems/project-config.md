---
trigger: always_on
description: Ria Living Systems is an ambitious all-in-one business management platform with 10+ integrated modules. This document provides guidelines to maintain clarity and prevent architectural debt as the platform grows.
---

# Claude Development Guidelines for Ria Living Systems

## Project Context
Ria Living Systems is an ambitious all-in-one business management platform with 10+ integrated modules. This document provides guidelines to maintain clarity and prevent architectural debt as the platform grows.

## Core Principles

### 1. Module Boundaries Are Sacred
- **NEVER** add direct imports between feature modules
- **ALWAYS** use the shared packages (`@ria/client`, `@ria/utils`) for cross-module communication
- **PREFER** event-driven patterns over direct coupling between modules

### 2. Follow the Clean Architecture Pattern
- **Repository Pattern** for ALL data access - no direct API calls in components
- **Zustand Stores** for ALL state management - no useState for shared data
- **Component Library** for ALL UI elements - no inline styles or custom components
- **Error Boundaries** for ALL pages - graceful error handling everywhere
- **TypeScript-first** - no `any` types, proper interfaces for all data structures

### 3. Maintain the Modular Monolith Architecture
```
apps/
  web/          → Next.js frontend (pages only, no business logic)
  api/          → NestJS backend (controllers only, delegates to packages)
  workers/      → Background jobs (uses packages for logic)
  collab/       → Real-time features (WebSocket handling)

packages/
  [module]-server/  → Business logic for each module
  web-ui/          → Shared UI components (ALL UI COMPONENTS MUST LIVE HERE)
  db/              → Database schema and migrations
  client/          → Shared client utilities
  utils/           → Common helpers
```

### 4. Component Library is Mandatory
- **NEVER** create UI components directly in apps/web
- **ALWAYS** define components in packages/web-ui first
- **IMPORT** all components from `@ria/web-ui`
- **CHECK** COMPONENT_LIBRARY.md before creating new components
- **REUSE** existing components - check if something similar exists

### 5. Code Organization is Non-Negotiable
- **FOLLOW** CODE_ORGANIZATION.md for ALL code placement decisions
- **HOOKS** must live in `@ria/web-hooks` - no custom hooks elsewhere
- **UTILITIES** must live in `@ria/utils` - no duplicate helper functions
- **BUSINESS LOGIC** must live in `@ria/[module]-server` packages
- **API CLIENTS** must live in `@ria/client` - centralized API communication
- **TYPES** that are shared must live in `@ria/types` or be co-located
- **NEVER** duplicate code - if you write it twice, you're doing it wrong

### 6. Routing Must Be Centralized
- **ALWAYS** use `ROUTES` constants from `@ria/utils/routes` - never hardcode paths
- **NEVER** use string literals like `"/auth/sign-in"` for routing
- **UPDATE** `@ria/utils/routes.ts` when adding new routes
- **EXAMPLE**: Use `ROUTES.SIGN_IN` instead of `"/auth/sign-in"`
- **REASON**: Base URLs can change, deployments can be under subpaths

### 7. No Emojis in Production Code
- **NEVER** use emojis in component text, labels, or user-facing content
- **AVOID** emojis in comments, function names, or variable names  
- **USE** proper icons from design system instead of emoji shortcuts
- **EXCEPTION**: Demo/placeholder data can use emojis temporarily
- **REASON**: Accessibility, professionalism, cross-platform compatibility

## Clean Architecture Implementation

### Architecture Layers
```
┌──────────────────────────────────┐
│     UI Components (@ria/web-ui)   │ ← Reusable components only
├──────────────────────────────────┤
│      Stores (@ria/client)         │ ← State management (Zustand)
├──────────────────────────────────┤
│   Repositories (@ria/client)      │ ← Data access layer
├──────────────────────────────────┤
│       API Client (@ria/client)    │ ← HTTP/WebSocket communication
├──────────────────────────────────┤
│    Database Schema (@ria/db)      │ ← Prisma ORM
└──────────────────────────────────┘
```

### Repository Pattern (MANDATORY)
```typescript
// ❌ WRONG - Direct API calls in components
const MyComponent = () => {
  const [data, setData] = useState([]);
  useEffect(() => {
    fetch('/api/items').then(res => res.json()).then(setData);
  }, []);
};

// ✅ CORRECT - Use repository pattern
// In packages/client/src/repositories/items.repository.ts
export class ItemsRepository extends BaseRepository<Item> {
  protected endpoint = '/items';
  
  async getActiveItems(): Promise<Item[]> {
    return this.request('GET', '/active');
  }
}

// In component - use store that uses repository
const MyComponent = () => {
  const { items, fetchItems } = useItemsStore();
  useEffect(() => {
    fetchItems();
  }, []);
};
```

### State Management with Zustand (MANDATORY)
```typescript
// ❌ WRONG - Local state for shared data
const [documents, setDocuments] = useState([]);
const [loading, setLoading] = useState(false);

// ✅ CORRECT - Use Zustand store
export const useDocumentsStore = create<DocumentsStore>()(
  devtools(
    immer((set, get) => ({
      documents: [],
      loading: false,
      error: null,
      
      fetchDocuments: async () => {
        set(state => { state.loading = true; });
        try {
          const response = await documentsRepository.findAll();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Askolnick) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
