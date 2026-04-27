---
trigger: always_on
description: typescript-guideline
---

# 📘 TypeScript Best Practices Guidelines

This document outlines TypeScript best practices for the AllInOne React Native project, focusing on type safety, maintainability, and developer experience.

## 🎯 Core TypeScript Principles

### 1. Type Safety First
Leverage TypeScript's type system to catch errors at compile time, not runtime.

### 2. Explicit Over Implicit
Be explicit about types when it improves code clarity and maintainability.

### 3. Progressive Enhancement
Start with basic types and gradually add more sophisticated typing as needed.

## 🔧 Configuration Best Practices

### tsconfig.json Setup
```json
{
  "compilerOptions": {
    "target": "es2020",
    "lib": ["es2020", "dom"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-native"
  },
  "include": [
    "src/**/*",
    "types/**/*"
  ],
  "exclude": [
    "node_modules",
    "android",
    "ios"
  ]
}
```

## 🏗️ Interface & Type Definitions

### Interface Best Practices
```typescript
// Use PascalCase for interfaces
interface UserProfile {
  readonly id: string;
  name: string;
  email: string;
  avatar?: string; // Optional properties with ?
  createdAt: Date;
  settings: UserSettings;
}

// Extend interfaces for related types
interface AdminProfile extends UserProfile {
  permissions: Permission[];
  role: AdminRole;
}

// Use generic interfaces for reusability
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
  errors?: string[];
}
```

### Type Aliases vs Interfaces
```typescript
// Use type aliases for unions, primitives, and computed types
type Status = 'loading' | 'success' | 'error';
type ID = string | number;
type EventHandler<T> = (event: T) => void;

// Use interfaces for object shapes that might be extended
interface ComponentProps {
  title: string;
  onPress: () => void;
}

// Prefer interfaces for React component props
interface ButtonProps {
  title: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onPress: () => void;
}
```

## 🎨 React Native Specific Types

### Component Props Typing
```typescript
import { ReactNode } from 'react';
import { ViewStyle, TextStyle } from 'react-native';

interface CustomButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  loading?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
  children?: ReactNode;
}

const CustomButton: React.FC<CustomButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  disabled = false,
  loading = false,
  style,
  textStyle,
  children
}) => {
  // Component implementation
};
```

### Navigation Types
```typescript
// Define navigation param lists
type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: undefined;
  TransactionDetail: { transactionId: string };
};

type BottomTabParamList = {
  Home: undefined;
  Investments: undefined;
  Reports: undefined;
};

// Use with navigation hooks
import { NavigationProp, RouteProp } from '@react-navigation/native';

type ProfileScreenNavigationProp = NavigationProp<RootStackParamList, 'Profile'>;
type ProfileScreenRouteProp = RouteProp<RootStackParamList, 'Profile'>;

interface ProfileScreenProps {
  navigation: ProfileScreenNavigationProp;
  route: ProfileScreenRouteProp;
}
```

### Redux/State Types
```typescript
// Define state interfaces
interface AuthState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
}

interface TransactionState {
  transactions: Transaction[];
  isLoading: boolean;
  filter: TransactionFilter;
}

// Root state type
interface RootState {
  auth: AuthState;
  transactions: TransactionState;
  investments: InvestmentState;
}

// Action types
interface LoginAction {
  type: 'auth/login';
  payload: { email: string; password: string };
}

interface LoginSuccessAction {
  type: 'auth/loginSuccess';
  payload: User;
}

type AuthAction = LoginAction | LoginSuccessAction;
```

## 🔥 Advanced TypeScript Patterns

### Utility Types Usage
```typescript
// Pick - Select specific properties
type UserPreview = Pick<User, 'id' | 'name' | 'avatar'>;

// Omit - Exclude specific properties
type CreateUserRequest = Omit<User, 'id' | 'createdAt'>;

// Partial - Make all properties optional
type UpdateUserRequest = Partial<Pick<User, 'name' | 'email' | 'avatar'>>;

// Required - Make all properties required
type CompleteUserProfile = Required<UserProfile>;

// Record - Create object type with specific keys
type TransactionsByCategory = Record<TransactionCategory, Transaction[]>;
```

### Conditional Types
```typescript
// Create conditional types for API responses
type ApiResult<T> = T extends string 
  ? { message: T }
  : { data: T };

// Conditional props based on variant
type ButtonProps<T extends 'primary' | 'secondary'> = {
  variant: T;
  title: string;
} & (T extends 'primary' 
  ? { color?: never } 
  : { color: string });
```

### Generic Constraints
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goktugoner23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
