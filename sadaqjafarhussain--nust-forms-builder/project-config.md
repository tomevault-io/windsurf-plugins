---
trigger: always_on
description: - `apps/web/` - Main Next.js web application
---

# Formbricks Architecture & Patterns

## Monorepo Structure

### Apps Directory
- `apps/web/` - Main Next.js web application
- `packages/` - Shared packages and utilities

### Key Directories in Web App
```
apps/web/
├── app/                    # Next.js 13+ app directory
│   ├── (app)/             # Main application routes
│   ├── (auth)/            # Authentication routes
│   ├── api/               # API routes
├── components/            # Shared components
├── lib/                   # Utility functions and services
└── modules/               # Feature-specific modules
```

## Routing Patterns

### App Router Structure
The application uses Next.js 13+ app router with route groups:

```
(app)/environments/[environmentId]/
├── surveys/[surveyId]/
│   ├── (analysis)/        # Analysis views
│   │   ├── responses/     # Response management
│   │   ├── summary/       # Survey summary
│   │   └── hooks/         # Analysis-specific hooks
│   ├── edit/              # Survey editing
│   └── settings/          # Survey settings
```

### Dynamic Routes
- `[environmentId]` - Environment-specific routes
- `[surveyId]` - Survey-specific routes

## Service Layer Pattern

### Service Organization
Services are organized by domain in `apps/web/lib/`:

```typescript
// Example: Response service
// apps/web/lib/response/service.ts
export const getResponseCountAction = async ({
  surveyId,
  filterCriteria,
}: {
  surveyId: string;
  filterCriteria: any;
}) => {
  // Service implementation
};
```

### Action Pattern
Server actions follow a consistent pattern:

```typescript
// Action wrapper for service calls
export const getResponseCountAction = async (params) => {
  try {
    const result = await responseService.getCount(params);
    return { data: result };
  } catch (error) {
    return { error: error.message };
  }
};
```

## Context Patterns

### Provider Structure
Context providers follow a consistent pattern:

```typescript
// Provider component
export const ResponseFilterProvider = ({ children }: { children: React.ReactNode }) => {
  const [selectedFilter, setSelectedFilter] = useState(defaultFilter);
  
  const value = {
    selectedFilter,
    setSelectedFilter,
    // ... other state and methods
  };

  return (
    <ResponseFilterContext.Provider value={value}>
      {children}
    </ResponseFilterContext.Provider>
  );
};

// Hook for consuming context
export const useResponseFilter = () => {
  const context = useContext(ResponseFilterContext);
  if (!context) {
    throw new Error('useResponseFilter must be used within ResponseFilterProvider');
  }
  return context;
};
```

### Context Composition
Multiple contexts are often composed together:

```typescript
// Layout component with multiple providers
export default function AnalysisLayout({ children }: { children: React.ReactNode }) {
  return (
    <ResponseFilterProvider>
      <ResponseCountProvider>
        {children}
      </ResponseCountProvider>
    </ResponseFilterProvider>
  );
}
```

## Component Patterns

### Page Components
Page components are located in the app directory and follow this pattern:

```typescript
// apps/web/app/(app)/environments/[environmentId]/surveys/[surveyId]/(analysis)/responses/page.tsx
export default function ResponsesPage() {
  return (
    <div>
      <ResponsesTable />
      <ResponsesPagination />
    </div>
  );
}
```

### Component Organization
- **Pages** - Route components in app directory
- **Components** - Reusable UI components
- **Modules** - Feature-specific components and logic

### Shared Components
Common components are in `apps/web/components/`:
- UI components (buttons, inputs, modals)
- Layout components (headers, sidebars)
- Data display components (tables, charts)

## Hook Patterns

### Custom Hook Structure
Custom hooks follow consistent patterns:

```typescript
export const useResponseCount = ({ 
  survey, 
  initialCount 
}: {
  survey: TSurvey;
  initialCount?: number;
}) => {
  const [responseCount, setResponseCount] = useState(initialCount ?? 0);
  const [isLoading, setIsLoading] = useState(false);
  
  // Hook logic...
  
  return {
    responseCount,
    isLoading,
    refetch,
  };
};
```

### Hook Dependencies
- Use context hooks for shared state
- Implement proper cleanup with AbortController
- Optimize dependency arrays to prevent unnecessary re-renders

## Data Fetching Patterns

### Server Actions
The app uses Next.js server actions for data fetching:

```typescript
// Server action
export async function getResponsesAction(params: GetResponsesParams) {
  const responses = await getResponses(params);
  return { data: responses };
}

// Client usage
const { data } = await getResponsesAction(params);
```

### Error Handling
Consistent error handling across the application:

```typescript
try {
  const result = await apiCall();
  return { data: result };
} catch (error) {
  console.error("Operation failed:", error);
  return { error: error.message };
}
```

## Type Safety

### Type Organization
Types are organized in packages:
- `@formbricks/types` - Shared type definitions
- Local types in component/hook files

### Common Types
```typescript
import { TSurvey } from "@formbricks/types/surveys/types";
import { TResponse } from "@formbricks/types/responses";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SadaqJafarHussain/NUST-Forms-Builder](https://github.com/SadaqJafarHussain/NUST-Forms-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
