---
trigger: always_on
description: Guidelines for AI agents working on the AniList Bulk Edit codebase.
---

# AGENTS.md

Guidelines for AI agents working on the AniList Bulk Edit codebase.

## Build/Lint/Test Commands

```bash
# Development
npm run dev          # Start development server on localhost:3000
npm run build        # Build production application
npm run start        # Start production server

# Code Quality
npm run lint         # Run ESLint (Next.js config)
npm run type-check   # Run TypeScript type checking (tsc --noEmit)

# Pre-commit checklist
npm run lint && npm run type-check
```

**Note:** No test framework is currently configured. When adding tests, consider Jest with React Testing Library.

## Project Structure

```
src/
  components/     # React components (PascalCase filenames)
  contexts/       # React context providers
  lib/            # Utilities, API clients, helpers (camelCase filenames)
  pages/          # Next.js pages and API routes
  store/          # Zustand store definitions
  types/          # TypeScript type definitions
```

## Code Style Guidelines

### Imports

Use path aliases defined in `tsconfig.json`:

```typescript
// Path aliases (preferred)
import { User, MediaList } from '@/types/anilist'
import { useStore } from '@/store'
import { AniListClient } from '@/lib/anilist'
import BulkEditPanel from '@/components/BulkEditPanel'

// Import order:
// 1. External packages (react, next, third-party)
// 2. Internal types (@/types/*)
// 3. Internal utilities (@/lib/*)
// 4. Store (@/store/*)
// 5. Components (@/components/*)
```

### Formatting

- **Indentation:** 4 spaces
- **Quotes:** Single quotes for imports, double quotes in JSX attributes
- **Semicolons:** Optional (codebase has mixed usage, be consistent within files)
- **Line length:** No strict limit, but keep readable (~100-120 chars)
- **Trailing commas:** Use in multiline arrays/objects

### TypeScript

TypeScript strict mode is enabled. Follow these patterns:

```typescript
// Interfaces for object shapes
export interface User {
    id: number;
    name: string;
    avatar?: {           // Optional properties use ?
        large?: string;
        medium?: string;
    };
}

// Enums for fixed value sets
export enum MediaListStatus {
    CURRENT = 'CURRENT',
    PLANNING = 'PLANNING',
    COMPLETED = 'COMPLETED',
    DROPPED = 'DROPPED',
    PAUSED = 'PAUSED',
    REPEATING = 'REPEATING'
}

// Record for dynamic key objects
customLists?: Record<string, boolean>;

// Type unions for specific values
type SortOrder = 'asc' | 'desc'

// Generic constraints
async execute<T>(requestFn: () => Promise<T>): Promise<T>
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| React Components | PascalCase | `BulkEditPanel`, `MediaListView` |
| Component files | PascalCase.tsx | `BulkEditPanel.tsx` |
| Utility files | camelCase.ts | `rateLimiter.ts`, `anilist.ts` |
| Interfaces/Types | PascalCase | `MediaList`, `FilterOptions` |
| Enums | PascalCase | `MediaType`, `ScoreFormat` |
| Functions/Methods | camelCase | `getCurrentUser()`, `applyFilters()` |
| Variables | camelCase | `selectedEntries`, `isLoading` |
| Constants | camelCase or UPPER_SNAKE | `maxRetries`, `ANILIST_API_URL` |
| CSS classes | kebab-case (Tailwind) | `bg-blue-500`, `dark:text-white` |

### React Components

```typescript
// Props interface above component
interface BulkEditPanelProps {
    client: AniListClient | null
}

// Functional component with explicit return
export default function BulkEditPanel({ client }: BulkEditPanelProps) {
    // Hooks at top
    const { selectedEntries, bulkEditMode } = useStore()
    const [isProcessing, setIsProcessing] = useState(false)

    // Event handlers
    const handleBulkOperation = async () => {
        // ...
    }

    // Early returns for conditional rendering
    if (!bulkEditMode) {
        return <div>...</div>
    }

    return <div>...</div>
}
```

### State Management (Zustand)

```typescript
// Use the store hook with destructuring
const { user, setUser, addNotification } = useStore()

// Access store outside React components
useStore.getState().fetchMediaLists(userId, type, true)

// State updates via actions only
setBulkEditMode(true)
addNotification({ type: 'success', message: 'Done!' })
```

### Error Handling

```typescript
// API calls with proper error handling
try {
    const result = await client.updateMediaListEntry(mediaId, updates)
    addNotification({ type: 'success', message: 'Updated!' })
} catch (error: any) {
    console.error('Update failed:', error)
    
    // Handle specific error types
    if (error.status === 401) {
        // Authentication error
    } else if (error.status === 429) {
        // Rate limit - implement backoff
    }
    
    addNotification({ type: 'error', message: error.message || 'Failed' })
}

// Exponential backoff for retries
const delay = Math.pow(2, retryCount) * 1000
await new Promise(resolve => setTimeout(resolve, delay))
```

### API Routes (Next.js)

```typescript
// src/pages/api/auth/signin.ts
import { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
    // Method validation
    if (req.method !== 'POST') {
        return res.status(405).json({ error: 'Method not allowed' })
    }

    // Input validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ari-03/Anilist-Bluk-Edit](https://github.com/Ari-03/Anilist-Bluk-Edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
