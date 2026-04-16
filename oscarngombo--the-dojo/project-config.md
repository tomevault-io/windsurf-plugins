---
trigger: always_on
description: The system follows a hierarchical structure with role-based access control, focusing on managing trainees, training subjects (courses), and assigned tasks (assignments/projects).
---

# The Dojo - Copilot Instructions

## Project Overview: Training Management Platform

The system follows a hierarchical structure with role-based access control, focusing on managing trainees, training subjects (courses), and assigned tasks (assignments/projects).

## Technology Stack (STRICT REQUIREMENTS)

### Core Technologies ONLY

- **React 19+** with **TypeScript** (strict mode)
- **TanStack Router** - File-based routing only
- **Native Fetch API** - HTTP requests
- **React Context API** - State management via Provider Pattern
- **CSS Modules or inline styles** - No external CSS-in-JS libraries

### Prohibited External Libraries

- ❌ No state management libraries (Redux, Zustand, MobX, etc.)
- ❌ No UI component libraries (Material-UI, Chakra, Ant Design, etc.)
- ❌ No toast/notification libraries
- ❌ No CSS-in-JS libraries (styled-components, emotion, etc.)
- ❌ No utility libraries (lodash, moment, date-fns, etc.)
- ❌ No form libraries (Formik, React Hook Form, etc.)

### TypeScript Requirements

- Strict mode enabled in `tsconfig.json`
- All components must have proper TypeScript interfaces
- API response types must be defined
- No `any` types allowed - use `unknown` if necessary
- All props interfaces must be explicitly defined

## Core System Architecture

### Backend Integration

- **RESTful API** with JSON responses (see Postman collection for endpoints `Resources/postman/`)
- **HTTP Methods**: GET, POST, PUT, DELETE
- **Error Handling**: Standardized error responses with status codes
- **Authentication**: Bearer token-based (JWT)
- **Data Storage**: Relational database
- **Security**: CORS-enabled with encrypted authentication

### File Structure

- create a clear, feature-based file structure below: is a guide but only create folders that are necessary for the project

```
src/
├── routes/           # TanStack Router file-based routes
├── components/       # Reusable UI components
├── providers/        # Context providers
├── hooks/           # Custom hooks
├── api/             # API service layer
├── types/           # TypeScript type definitions
├── utils/           # Utility functions
└── styles/          # CSS modules
```

## Provider Pattern Implementation

### Core Provider Structure

```typescript
// providers/AppProvider.tsx
interface AppContextType {
  state: AppState;
  actions: AppActions;
}

interface AppState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

interface AppActions {
  setUser: (user: User) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
}

const AppContext = createContext<AppContextType | undefined>(undefined);

export const AppProvider: React.FC<{children: React.ReactNode}> = ({children}) => {
  const [state, setState] = useState<AppState>(initialState);

  const actions = useMemo<AppActions>(() => ({
    setUser: (user) => setState(prev => ({...prev, user})),
    setLoading: (loading) => setState(prev => ({...prev, loading})),
    setError: (error) => setState(prev => ({...prev, error}))
  }), []);

  return (
    <AppContext.Provider value={{state, actions}}>
      {children}
    </AppContext.Provider>
  );
};

export const useApp = (): AppContextType => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
};
```

### Custom Toast Provider

```typescript
// providers/ToastProvider.tsx
interface Toast {
  id: string
  message: string
  type: 'success' | 'error' | 'warning' | 'info'
  duration?: number
}

interface ToastContextType {
  toasts: Toast[]
  addToast: (toast: Omit<Toast, 'id'>) => void
  removeToast: (id: string) => void
}

export const ToastProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  // Implementation here
}

export const useToast = (): ToastContextType => {
  // Hook implementation
}
```

## Key System Components

### 1. User Management System

- **Multi-role architecture**: Admin and Trainee roles
- **Approval workflow**: Trainees can be approved, rejected, or kept pending
- **Profile management**: Basic user profiles (reference Postman collection for fields)
- **Account lifecycle**: Full CRUD operations for user accounts

### 2. Subject/Course Management

- **Curriculum structure**: Subjects represent training modules/courses
- **Content organization**: Each subject has detailed descriptions and metadata
- **Active/inactive status**: Subjects can be enabled/disabled
- **Audit trail**: Tracks who created/modified subjects

### 3. Task/Assignment Management

- **Structured assignments**: Tasks are linked to specific subjects
- **Detailed requirements**: Each task includes description, requirements, and scoring
- **Time management**: Due dates and deadlines
- **Scoring system**: Maximum scores for evaluation
- **Status tracking**: Tasks can be active/inactive

## Data Relationships & Types

```typescript
// types/index.ts
interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'trainee'
  status: 'approved' | 'pending' | 'rejected'
  avatar?: string
  createdAt: string
  updatedAt: string
}

interface Subject {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OscarNgombo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
