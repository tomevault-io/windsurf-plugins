---
trigger: always_on
description: This project follows specific patterns for component development, state management, and UI composition. Always use the established patterns and Plop generators.
---

# Frontend Development Instructions

## Overview

This project follows specific patterns for component development, state management, and UI composition. Always use the established patterns and Plop generators.

## Component Architecture

### 1. Component Types

**UI Components** (`src/ui/`):
- Pure, reusable components
- No business logic
- Flat structure (no nested folders)

**Route Components** (`src/routes/`):
- Page-level components
- Can contain business logic
- Organized by route structure

**Container Components**:
- Wrap UI components with data fetching logic
- Handle state management for specific features

### 2. Component Development Patterns

**UI Component Structure:**
```typescript
// src/ui/button/Button.tsx
import { ButtonProps } from './Button.types';
import './Button.css';

export const Button = ({ variant = 'primary', children, ...props }: ButtonProps) => {
  return (
    <button 
      className={`button button--${variant}`} 
      {...props}
    >
      {children}
    </button>
  );
};
```

**Component Types:**
```typescript
// src/ui/button/Button.types.ts
import { ButtonHTMLAttributes, ReactNode } from 'react';

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  children: ReactNode;
}
```

**Component Tests:**
```typescript
// src/ui/button/Button.test.tsx
import { render, screen } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });

  it('applies variant class', () => {
    render(<Button variant="secondary">Test</Button>);
    expect(screen.getByRole('button')).toHaveClass('button--secondary');
  });
});
```

### 3. Route Component Patterns

**Route Component Structure:**
```typescript
// src/routes/users/-components/UsersList.tsx
import { useQuery } from 'hooks/useQuery/useQuery';
import { UserCard } from 'ui/userCard/UserCard';
import { Loader } from 'ui/loader/Loader';

export const UsersList = () => {
  const { data: users, isLoading, error } = useQuery('getUsers');

  if (isLoading) return <Loader />;
  if (error) return <div>Error loading users</div>;

  return (
    <div className="users-list">
      <h2>Users</h2>
      <div className="users-grid">
        {users?.map(user => (
          <UserCard key={user.id} user={user} />
        ))}
      </div>
    </div>
  );
};
```

**Route with Data Loading:**
```typescript
// src/routes/users/user.$userId.tsx
import { createFileRoute } from '@tanstack/react-router';
import { UserProfile } from './-components/UserProfile';

export const Route = createFileRoute('/users/$userId')({
  component: UserProfile,
  loader: ({ params }) => {
    // Pre-load data if needed
    return { userId: params.userId };
  },
});
```

## Custom Hooks

### 1. Hook Development Patterns

**Data Fetching Hook:**
```typescript
// src/hooks/useUserProfile/useUserProfile.tsx
import { useQuery } from 'hooks/useQuery/useQuery';
import { useParams } from '@tanstack/react-router';

export const useUserProfile = () => {
  const { userId } = useParams({ from: '/users/$userId' });
  
  const { data: user, isLoading, error } = useQuery('getUser', {
    variables: userId,
    enabled: !!userId,
  });

  return {
    user,
    isLoading,
    error,
    isLoggedInUser: user?.id === 'current-user-id', // Example derived state
  };
};
```

**Hook with Local State:**
```typescript
// src/hooks/useToggle/useToggle.tsx
import { useState, useCallback } from 'react';

export const useToggle = (initialValue: boolean = false) => {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(prev => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return {
    value,
    toggle,
    setTrue,
    setFalse,
    setValue,
  };
};
```

**Hook Tests:**
```typescript
// src/hooks/useToggle/useToggle.test.tsx
import { renderHook, act } from '@testing-library/react';
import { useToggle } from './useToggle';

describe('useToggle', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current.value).toBe(false);
  });

  it('toggles value correctly', () => {
    const { result } = renderHook(() => useToggle());
    
    act(() => {
      result.current.toggle();
    });
    
    expect(result.current.value).toBe(true);
  });
});
```

### 2. Hook Export Pattern

Always export hooks from the main hooks index:

```typescript
// src/hooks/index.ts
export * from './useToggle/useToggle';
export * from './useUserProfile/useUserProfile';
export * from './useLocalStorage/useLocalStorage';
```

## Styling Guidelines

### 1. CSS Modules Pattern

```css
/* src/ui/button/Button.css */
.button {
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.button--primary {
  background-color: #007bff;
  color: white;
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheSoftwareHouse/react-starter-boilerplate](https://github.com/TheSoftwareHouse/react-starter-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
