---
trigger: always_on
description: TypeScript best practices, patterns, and type system mastery for modern applications
---

# TypeScript Best Practices

Comprehensive guide for TypeScript development with focus on type safety, performance, and maintainability.

## 1. Type System Mastery

### 1.1 Type Inference and Annotations

```typescript
// Let TypeScript infer when obvious
const numbers = [1, 2, 3]; // number[]
const config = { host: 'localhost', port: 3000 }; // inferred shape

// Annotate when inference isn't sufficient
const processValue = <T extends { id: string }>(value: T): T & { processed: true } => {
  return { ...value, processed: true };
};

// Use satisfies for type checking without widening
const routes = {
  home: '/',
  users: '/users',
  profile: '/users/:id'
} satisfies Record<string, string>;
```

### 1.2 Discriminated Unions and Type Narrowing

```typescript
// Discriminated unions for state management
type AsyncState<T> = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Type guards for narrowing
function isSuccess<T>(state: AsyncState<T>): state is Extract<AsyncState<T>, { status: 'success' }> {
  return state.status === 'success';
}

// Pattern matching with exhaustive checks
function handleState<T>(state: AsyncState<T>): string {
  switch (state.status) {
    case 'idle': return 'Ready';
    case 'loading': return 'Loading...';
    case 'success': return `Data: ${JSON.stringify(state.data)}`;
    case 'error': return `Error: ${state.error.message}`;
    default: {
      const _exhaustive: never = state;
      return _exhaustive;
    }
  }
}
```

### 1.3 Advanced Type Patterns

```typescript
// Conditional types
type IsArray<T> = T extends readonly any[] ? true : false;
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

// Mapped types with modifiers
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Template literal types
type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type APIEndpoint<M extends HTTPMethod> = `/api/${Lowercase<M>}/${string}`;

// Branded types for nominal typing
type UserId = string & { __brand: 'UserId' };
type PostId = string & { __brand: 'PostId' };

const createUserId = (id: string): UserId => id as UserId;
```

### 1.4 Utility Type Cookbook

```typescript
// Deep partial with arrays
type DeepPartial<T> = T extends readonly any[] ? T : {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Type-safe object paths
type Path<T> = T extends object ? {
  [K in keyof T]: K extends string 
    ? T[K] extends object 
      ? K | `${K}.${Path<T[K]>}`
      : K
    : never;
}[keyof T] : never;

// Type-safe event emitter
type EventMap = Record<string, any>;
type EventKey<T extends EventMap> = string & keyof T;
type EventReceiver<T> = (params: T) => void;

interface Emitter<T extends EventMap> {
  on<K extends EventKey<T>>(eventName: K, fn: EventReceiver<T[K]>): void;
  emit<K extends EventKey<T>>(eventName: K, params: T[K]): void;
}
```

## 2. Strict Mode Best Practices

### 2.1 Null Safety

```typescript
// Handle null/undefined explicitly
interface User {
  id: string;
  name: string;
  email?: string; // Optional
  metadata: Record<string, unknown> | null; // Nullable
}

// Non-null assertion when you're certain
function processUser(user: User | null) {
  if (!user) throw new Error('User required');
  
  // TypeScript knows user is non-null here
  console.log(user.name.toUpperCase());
  
  // Optional chaining for safety
  const domain = user.email?.split('@')[1];
}

// Nullish coalescing
const getDisplayName = (user: User) => {
  return user.name ?? user.email ?? 'Anonymous';
};
```

### 2.2 Type Assertions and Guards

```typescript
// Avoid type assertions, prefer type guards
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    typeof value.id === 'string' &&
    typeof value.name === 'string'
  );
}

// When assertions are necessary, be explicit
const config = JSON.parse(configString) as unknown;
if (!isValidConfig(config)) {
  throw new Error('Invalid configuration');
}
```

## 3. Architecture Patterns

### 3.1 Dependency Injection

```typescript
// Token-based DI
const TOKENS = {
  Logger: Symbol('Logger'),
  Database: Symbol('Database'),
  Cache: Symbol('Cache'),
} as const;

interface Logger {
  log(message: string): void;
}

interface Container {
  get<T>(token: symbol): T;
  bind<T>(token: symbol, factory: () => T): void;
}

// Usage with decorators
class UserService {
  constructor(
    @inject(TOKENS.Logger) private logger: Logger,
    @inject(TOKENS.Database) private db: Database
  ) {}
}
```

### 3.2 Repository Pattern

```typescript
// Generic repository interface
interface Repository<T, ID = string> {
  findById(id: ID): Promise<T | null>;
  findAll(filter?: Partial<T>): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: ID): Promise<void>;
}

// Type-safe implementation
class UserRepository implements Repository<User, string> {
  async findById(id: string): Promise<User | null> {
    const result = await db.query<User>('SELECT * FROM users WHERE id = ?', [id]);
    return result[0] || null;
  }
  
  async findAll(filter?: Partial<User>): Promise<User[]> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
