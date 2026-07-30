---
trigger: always_on
description: - Components over 300 lines
---

# Code Architecture & Reusability Guidelines

## CRITICAL ARCHITECTURE RULES

### NEVER CREATE:
- Components over 300 lines
- Duplicate logic across components
- Inline API calls in UI components
- Mixed concerns (UI + business logic + API)
- Copy-pasted code blocks

### ALWAYS CREATE:
- Single-responsibility components (20-100 lines)
- Custom hooks for reusable logic
- Service layer for API calls
- Shared types in `lib/types.ts`
- Composition patterns from smaller components

## Component Composition Over Monoliths

**NEVER create large, monolithic components that handle multiple concerns:**
**Avoid**: Single components or page that exceed 300 lines or handle multiple responsibilities
**Avoid**: Pages that contain all logic inline instead of using smaller components
**Avoid**: Components that mix UI rendering, business logic, and API calls

**ALWAYS break down complex functionality into smaller, reusable pieces:**
**Create**: Focused components with single responsibilities (20-100 lines)
**Create**: Composition patterns using multiple small components 
**Create**: Custom hooks for reusable logic extraction
**Create**: Separate layers for data fetching, business logic, and presentation

## Component Organization Strategy

### 1. Feature-Based Organization
```tsx
// Wrong: Everything in one file
const Dashboard = () => {
  // 300+ lines of mixed logic
  return <div>{/* massive JSX */}</div>
}

// ✅ Correct: Composed from focused components
const Dashboard = () => (
  <DashboardLayout>
    <DashboardHeader />
    <DashboardStats />
    <DashboardCharts />
    <DashboardActivity />
  </DashboardLayout>
)
```

### 2. Reusable Component Patterns
```tsx
// ✅ Base components for consistent UI patterns
const Card = ({ children, className, ...props }) => (
  <div className={cn("rounded-lg border bg-card", className)} {...props}>
    {children}
  </div>
)

// ✅ Composite components for complex patterns  
const StatsCard = ({ title, value, change, icon: Icon }) => (
  <Card className="p-6">
    <div className="flex items-center justify-between">
      <div>
        <p className="text-sm text-muted-foreground">{title}</p>
        <p className="text-2xl font-bold">{value}</p>
        <p className="text-sm text-green-600">{change}</p>
      </div>
      <Icon className="h-8 w-8 text-muted-foreground" />
    </div>
  </Card>
)
```

### 3. Logic Extraction Patterns
```tsx
// ✅ Custom hooks for reusable stateful logic
const useLocalStorage = (key: string, initialValue: any) => {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      return initialValue
    }
  })

  const setValue = (value: any) => {
    try {
      setStoredValue(value)
      window.localStorage.setItem(key, JSON.stringify(value))
    } catch (error) {
      console.error(error)
    }
  }

  return [storedValue, setValue]
}

// ✅ Service layer for API interactions
export const userService = {
  async getUser(id: string) {
    const response = await fetch(`/api/users/${id}`)
    return response.json()
  },
  
  async updateUser(id: string, data: Partial<User>) {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    })
    return response.json()
  }
}
```

## File Organization Best Practices

### 1. Component File Structure
```tsx
// ComponentName/index.ts - Export barrel
export { ComponentName } from './ComponentName'
export type { ComponentNameProps } from './ComponentName'

// ComponentName/ComponentName.tsx - Main component
interface ComponentNameProps {
  // Props definition
}

export const ComponentName = ({ ...props }: ComponentNameProps) => {
  // Component implementation
}

// ComponentName/ComponentName.stories.tsx - Storybook stories (if used)
// ComponentName/ComponentName.test.tsx - Tests (if used)
```

### 2. Shared Type Definitions
```tsx
// lib/types.ts - Shared application types
export interface User {
  id: string
  email: string
  name: string
  avatar?: string
}

export interface ApiResponse<T> {
  data: T
  message: string
  success: boolean
}

export type Theme = 'light' | 'dark' | 'system'
export type UserRole = 'admin' | 'user' | 'guest'
```

### 3. Utility Function Organization
```tsx
// lib/utils.ts - General utilities (keep existing cn function)
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

export function formatDate(date: Date | string, format?: string): string {
  // Date formatting utility
}

export function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  // Debounce utility
}

// lib/constants.ts - Application constants
export const API_ENDPOINTS = {
  USERS: '/api/users',
  POSTS: '/api/posts',
  AUTH: '/api/auth'
} as const

export const QUERY_KEYS = {
  USERS: 'users',
  POSTS: 'posts',
  USER_PROFILE: 'user-profile'
} as const
```

## Custom Hook Patterns for Reusability

### 1. Data Fetching Hooks
```tsx
// hooks/useUsers.ts
export const useUsers = () => {
  return useQuery({
    queryKey: [QUERY_KEYS.USERS],
    queryFn: () => userService.getAllUsers()
  })
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chihebnabil/lovable-boilerplate](https://github.com/chihebnabil/lovable-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
