---
trigger: always_on
description: api
---

# API Client & TypeScript Types

Bu proje modern API client architecture ve comprehensive TypeScript typing kullanır. Type safety, developer experience ve maintainability için optimize edilmiştir.

## API Client Architecture ([lib/api-client.ts](mdc:lib/api-client.ts))

### Modern API Client Class
```typescript
class APIClient {
  private baseURL: string;
  
  constructor(baseURL: string = '') {
    this.baseURL = baseURL;
  }

  // Generic request method with proper typing
  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    // Implementation with error handling
  }
}
```

### Organized API Functions
- **authAPI**: Authentication endpoints
- **usersAPI**: User management endpoints  
- **rolesAPI**: Role management endpoints
- **permissionsAPI**: Permission management endpoints

### Query Key Factories
```typescript
export const queryKeys = {
  auth: {
    currentUser: () => ['auth', 'currentUser'] as const,
    permissions: () => ['auth', 'permissions'] as const,
  },
  users: {
    all: () => ['users'] as const,
    list: (filters?: any) => ['users', 'list', filters] as const,
    detail: (id: string) => ['users', 'detail', id] as const,
  },
  // ...
};
```

## TypeScript Type System ([lib/types.ts](mdc:lib/types.ts))

### Core User Types
```typescript
export interface SimpleUser {
  id: string;
  name: string | null;
  email: string | null;
  permissions: string[];
  roles: SimpleRole[];
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface DetailedUser extends SimpleUser {
  profileImage: string | null;
  lastLoginAt: string | null;
  sessions: UserSession[];
}
```

### Role & Permission Types
```typescript
export interface SimpleRole {
  id: string;
  name: string;
  permissions: string[];
}

export interface Permission {
  id: string;
  name: string;
  description: string | null;
  category: string;
}
```

### API Response Types
```typescript
export interface APIResponse<T = any> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}

export interface PaginatedResponse<T> extends APIResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

### Form Data Types
```typescript
export interface LoginFormData {
  email: string;
  password: string;
  rememberMe?: boolean;
}

export interface CreateUserFormData {
  name: string;
  email: string;
  password: string;
  roleIds: string[];
}

export interface UpdateUserFormData {
  name?: string;
  email?: string;
  isActive?: boolean;
  roleIds?: string[];
}
```

### UI State Types
```typescript
export interface LoadingStates {
  login: boolean;
  logout: boolean;
  createUser: boolean;
  updateUser: boolean;
  deleteUser: boolean;
}

export interface ModalStates {
  createUser: boolean;
  editUser: boolean;
  deleteUser: boolean;
  changePassword: boolean;
}
```

## TanStack Query Integration

### Auth Hooks ([lib/hooks/useAuth.ts](mdc:lib/hooks/useAuth.ts))
```typescript
export const useCurrentUser = () => {
  return useQuery({
    queryKey: queryKeys.auth.currentUser(),
    queryFn: () => apiClient.authAPI.getCurrentUser(),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

export const useLogin = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: LoginFormData) => apiClient.authAPI.login(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.auth.currentUser() });
    },
  });
};
```

### User Management Hooks ([lib/hooks/useUsers.ts](mdc:lib/hooks/useUsers.ts))
```typescript
export const useUsers = (filters?: UserFilters) => {
  return useQuery({
    queryKey: queryKeys.users.list(filters),
    queryFn: () => apiClient.usersAPI.getUsers(filters),
  });
};

export const useCreateUser = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: CreateUserFormData) => apiClient.usersAPI.createUser(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.users.all() });
    },
  });
};
```

## Error Handling & Validation

### API Error Types
```typescript
export interface APIError {
  code: string;
  message: string;
  details?: Record<string, any>;
  statusCode: number;
}

export class APIClientError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string,
    public details?: Record<string, any>
  ) {
    super(message);
    this.name = 'APIClientError';
  }
}
```

### Form Validation Types
```typescript
export interface ValidationError {
  field: string;
  message: string;
}

export interface FormState<T> {
  data: T;
  errors: ValidationError[];
  isValid: boolean;
  isSubmitting: boolean;
}
```

## Constants & Configuration ([lib/constants.ts](mdc:lib/constants.ts))

### API Endpoints
```typescript
export const API_ENDPOINTS = {
  // Auth endpoints
  AUTH_LOGIN: "/api/auth/login",
  AUTH_LOGOUT: "/api/auth/logout",
  AUTH_CURRENT_USER: "/api/auth/current-user",
  
  // Admin endpoints
  ADMIN_USERS: "/api/admin/users",
  ADMIN_ROLES: "/api/admin/roles",
  ADMIN_PERMISSIONS: "/api/admin/permissions",
  
  // User endpoints
  USER_PROFILE: "/api/users/profile",
} as const;
```

### Permission Constants
```typescript
export const PERMISSIONS = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IamYGT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
