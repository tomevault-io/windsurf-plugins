---
trigger: always_on
description: TypeScript development patterns and best practices for the Medusa monorepo
---


# TypeScript Development Patterns

You are an expert in TypeScript, modern JavaScript, and type-safe development practices.

## Core TypeScript Principles

- Use strict TypeScript configuration
- Prefer type inference over explicit typing when clear
- Use union types and discriminated unions effectively
- Implement proper type guards and validation
- Leverage generic types for reusability
- Use `as const` for immutable data structures
- Prefer interfaces over type aliases for object shapes

## Type Definitions

### Interface Design
```typescript
// Use interfaces for object shapes
interface User {
  readonly id: string
  name: string
  email: string
  createdAt: Date
  updatedAt: Date
}

// Use generic interfaces for reusability
interface ApiResponse<T> {
  data: T
  success: boolean
  message?: string
}

// Extend interfaces for specialization
interface AdminUser extends User {
  role: "admin" | "super_admin"
  permissions: Permission[]
}
```

### Union Types and Discriminated Unions
```typescript
// Use discriminated unions for type safety
type PaymentStatus = 
  | { status: "pending"; pendingReason: string }
  | { status: "completed"; completedAt: Date }
  | { status: "failed"; error: string }

// Type guards for discriminated unions
function isCompletedPayment(
  payment: PaymentStatus
): payment is Extract<PaymentStatus, { status: "completed" }> {
  return payment.status === "completed"
}
```

### Generic Types
```typescript
// Generic utility types
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>
type RequiredFields<T, K extends keyof T> = T & Required<Pick<T, K>>

// Generic function types
type AsyncFunction<T extends any[], R> = (...args: T) => Promise<R>

// Generic class types
class Repository<T extends { id: string }> {
  async findById(id: string): Promise<T | null> {
    // Implementation
  }
  
  async create(data: Omit<T, "id">): Promise<T> {
    // Implementation
  }
}
```

## Advanced Type Patterns

### Conditional Types
```typescript
// Conditional types for API responses
type ApiResult<T> = T extends string 
  ? { message: T }
  : T extends Error
  ? { error: T }
  : { data: T }

// Mapped types for form validation
type ValidationErrors<T> = {
  [K in keyof T]?: string[]
}

// Template literal types
type EventName<T extends string> = `${T}:created` | `${T}:updated` | `${T}:deleted`
type UserEvents = EventName<"user"> // "user:created" | "user:updated" | "user:deleted"
```

### Utility Types
```typescript
// Custom utility types
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

type NonNullable<T> = T extends null | undefined ? never : T

type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
}

// Usage examples
type UserStringFields = PickByType<User, string> // { name: string; email: string }
type PartialUser = DeepPartial<User>
```

## Type Guards and Validation

### Runtime Type Checking
```typescript
// Type guards for runtime validation
function isString(value: unknown): value is string {
  return typeof value === "string"
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    "email" in value &&
    isString((value as any).id) &&
    isString((value as any).name) &&
    isString((value as any).email)
  )
}

// Assertion functions
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error("Value is not a valid User")
  }
}
```

### Zod Integration
```typescript
import { z } from "zod"

// Define schemas with Zod
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
  createdAt: z.date(),
  updatedAt: z.date(),
})

// Infer types from schemas
type User = z.infer<typeof UserSchema>

// Validation with proper error handling
function validateUser(data: unknown): User {
  const result = UserSchema.safeParse(data)
  
  if (!result.success) {
    throw new Error(`Invalid user data: ${result.error.message}`)
  }
  
  return result.data
}
```

## Error Handling Patterns

### Custom Error Types
```typescript
// Base error class
abstract class AppError extends Error {
  abstract readonly code: string
  abstract readonly statusCode: number
  
  constructor(message: string, public readonly context?: Record<string, any>) {
    super(message)
    this.name = this.constructor.name
  }
}

// Specific error types
class ValidationError extends AppError {
  readonly code = "VALIDATION_ERROR"
  readonly statusCode = 400
  
  constructor(
    message: string,
    public readonly errors: Record<string, string[]>
  ) {
    super(message)
  }
}

class NotFoundError extends AppError {
  readonly code = "NOT_FOUND"
  readonly statusCode = 404
}
```

### Result Pattern
```typescript
// Result type for error handling
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E }

// Helper functions
function success<T>(data: T): Result<T, never> {
  return { success: true, data }
}

function failure<E>(error: E): Result<never, E> {
  return { success: false, error }
}

// Usage in functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lambda-curry/medusa2-starter](https://github.com/lambda-curry/medusa2-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
