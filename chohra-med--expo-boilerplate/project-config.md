---
trigger: always_on
description: Feature-first architecture is a design pattern that organizes code by business features rather than technical layers. This approach promotes better maintainability, scalability, and team collaboration by keeping related functionality together and reducing coupling between different parts of the application.
---

# Feature-First Architecture Guide

## Overview

Feature-first architecture is a design pattern that organizes code by business features rather than technical layers. This approach promotes better maintainability, scalability, and team collaboration by keeping related functionality together and reducing coupling between different parts of the application.

## Core Principles

### 1. Feature Isolation
- Each feature is self-contained with its own components, logic, and state
- Features communicate through well-defined interfaces
- Minimal dependencies between features
- Clear boundaries and responsibilities

### 2. Shared Resources
- Common functionality is extracted to shared directories
- UI components, utilities, and services are globally accessible
- Consistent patterns across all features

### 3. Scalability
- Easy to add new features without affecting existing ones
- Clear structure for team members to work independently
- Simple to remove or refactor features

## Project Structure

```
src/
├── features/                    # Feature-specific code
│   ├── auth/                   # Authentication feature
│   │   ├── api/               # API calls and endpoints
│   │   ├── components/        # Feature-specific components
│   │   ├── hooks/            # Custom hooks
│   │   ├── screens/          # Screen components
│   │   ├── services/         # Business logic
│   │   ├── store/            # State management
│   │   └── types/            # Type definitions
│   ├── media-library/         # Media management feature
│   ├── collections/           # Collections feature
│   └── settings/              # Settings feature
├── navigation/                 # Navigation configuration
│   ├── navigators/           # Navigator components
│   ├── routes.ts             # Route definitions
│   └── routes.types.ts       # Navigation types
├── services/                  # Global services
│   ├── api/                  # API configuration
│   ├── datadog/              # Monitoring service
│   └── logging/              # Logging services
├── store/                     # Global store configuration
│   ├── store.ts              # Store setup
│   ├── reducers.ts           # Root reducer
│   └── app.slice.ts          # App-level state
├── ui/                        # Shared UI components
│   ├── components/           # Reusable components
│   ├── style/                # Theme and styling
│   └── tokens/               # Design tokens
├── utils/                     # Utility functions
├── schemas/                   # Data validation schemas
├── config/                    # Configuration files
└── entrypoints/              # App entry points
```

## Feature Structure Deep Dive

### 1. API Layer (`api/`)
```typescript
// features/auth/api/auth.api.ts
import { api } from "#root/services/api/api";
import { AuthSchema, type AuthResponse } from "./types";

const authApi = api.injectEndpoints({
  overrideExisting: true,
  endpoints: (builder) => ({
    login: builder.mutation<AuthResponse, LoginCredentials>({
      query: (credentials) => ({
        url: "/auth/login",
        method: "POST",
        body: credentials,
      }),
      responseSchema: AuthSchema,
    }),
  }),
});

export const { useLoginMutation } = authApi;
```

**Best Practices:**
- Use RTK Query for all API calls
- Define response schemas with Zod validation
- Type all request/response interfaces
- Use proper error handling
- Implement caching strategies

### 2. Components (`components/`)
```typescript
// features/auth/components/login-form.tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { loginSchema, type LoginFormData } from "../types";

export interface LoginFormProps {
  onSuccess: (data: LoginFormData) => void;
  onError: (error: string) => void;
}

export const LoginForm = ({ onSuccess, onError }: LoginFormProps) => {
  const { control, handleSubmit } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  // Implementation...
};
```

**Best Practices:**
- Clear prop interfaces with proper exports
- Use React.memo for performance optimization
- Implement proper form validation
- Follow single responsibility principle
- Use TypeScript template literal types for variants

### 3. Hooks (`hooks/`)
```typescript
// features/auth/hooks/use-auth.ts
import { useAppSelector, useAppDispatch } from "#root/store/store";
import { selectAuthState } from "../store/auth-selector";
import { login, logout } from "../store/auth-slice";

export const useAuth = () => {
  const dispatch = useAppDispatch();
  const authState = useAppSelector(selectAuthState);

  const handleLogin = useCallback(
    (credentials: LoginCredentials) => {
      dispatch(login(credentials));
    },
    [dispatch]
  );

  const handleLogout = useCallback(() => {
    dispatch(logout());
  }, [dispatch]);

  return {
    ...authState,
    login: handleLogin,
    logout: handleLogout,
  };
};
```

**Best Practices:**
- Encapsulate feature-specific logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chohra-med/expo_boilerplate](https://github.com/chohra-med/expo_boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
