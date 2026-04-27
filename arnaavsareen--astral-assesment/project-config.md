---
trigger: always_on
description: **Core Principle:** Write TypeScript and React code with absolute clarity as the top priority. Choose the most straightforward solution that any developer can understand at first glance.
---

# TypeScript & TSX Guidelines - v2

**Core Principle:** Write TypeScript and React code with absolute clarity as the top priority. Choose the most straightforward solution that any developer can understand at first glance.

**Development Philosophy:**

1. **Fewest lines, straightforward over clever** → Build only when needed
2. **One responsibility per function/component/module** → Single clear purpose
3. **Five-minute rule** → If you cannot explain it quickly, it is too complex
4. **Write for humans first** → Clever code is bad code

**Simple means:**

- Understandable in under 30 seconds
- Clear names: `calculateMonthlyPayment()` not `calc()`
- Max three nesting levels → Prefer early returns
- Numbered comments for multi-step flows: `// 1️⃣ Validate → // 2️⃣ Process → // 3️⃣ Return`

## Style Conventions

**TypeScript with these specifics:**

- Double quotes for strings
- Trailing commas in multi-line structures
- Type hints everywhere
- **Zod for all runtime validation** - Use schemas for uncertain data
- **Favor functions over classes** - Classes only for true stateful objects

**Naming Conventions:**

- `camelCase` for functions, variables, props
- `PascalCase` for components, types, interfaces
- `SCREAMING_SNAKE_CASE` for constants
- Prefix boolean props: `isLoading`, `hasError`, `canEdit`, `shouldShow`

```typescript
function createUserProfile(userData: CreateUserRequest, settings?: UserSettings, notifyAdmin: boolean = true): Promise<UserProfile> {
  // Implementation
}

const MAX_FILE_SIZE_MB = 10;
const ALLOWED_EXTENSIONS = [".jpg", ".png", ".gif", ".pdf"];

// Zod for runtime validation of uncertain inputs
const UserCreateRequestSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().min(13).max(120),
});

type UserCreateRequest = z.infer<typeof UserCreateRequestSchema>;
```

## File Structure

**1. Header (Required)**

```typescript
/* ==========================================================================*/
// userService.ts — User management and authentication operations
/* ==========================================================================*/
// Purpose: Handle user CRUD operations and session management
// Sections: Imports, Types, Functions, Public API
/* ==========================================================================*/
```

**2. Imports (Four groups)**

```typescript
/* ==========================================================================*/
// Imports
/* ==========================================================================*/

// Standard Library/Runtime --------------------------------------------------
import fs from "fs";
import path from "path";
import { EventEmitter } from "events";

// Third Party ---------------------------------------------------------------
import { z } from "zod";
import axios from "axios";
import express from "express";

// Core (App-wide) -----------------------------------------------------------
import { DatabaseConnection } from "@/lib/database";
import { logger } from "@/lib/logger";
import { AppError } from "@/lib/errors";
import { User, UserProfile } from "@/types/user";

// Internal (Current Module) -------------------------------------------------
import { validateEmail } from "./helpers";
import { formatUserData } from "./utils";
```

**3. Types (Component-specific only)**

```typescript
/* ==========================================================================*/
// Types (Component-specific only)
/* ==========================================================================*/

// Only include types that are hyper-specific to this file
interface UserProfileProps {
  userId: string;
  editable?: boolean;
  onUpdate?: (user: User) => void;
}

// App-wide types should be in dedicated @/types files
```

**4. Public API - Be Intentional**

```typescript
/* ==========================================================================*/
// Public API
/* ==========================================================================*/

// Don't re-export everything - be intentional about what's public
export { createUser, getUserById }; // Only these are part of public API
export type { CreateUserRequest }; // Only export types others need

// Keep internal - don't export helpers, utilities, or implementation details
// validateEmail, formatUserData, etc. stay private to this module
```

## Error Handling - Fail Fast & Simple

**Core Principles:**

1. **Validate only uncertain inputs** - API responses, user input, external data
2. **Don't re-validate TypeScript-guaranteed values** - Trust your types
3. **Catch only what you can handle** - Let other exceptions bubble up
4. **One try/catch per function max** - If you need more, refactor
5. **Structured logging with context** - Preserve error causes and original stack traces
6. **Consider Result types for predictable failures** - Reserve exceptions for truly exceptional cases

```typescript
// Custom error class that preserves cause
class AppError extends Error {
  constructor(message: string, options?: { cause?: Error; userId?: string; context?: Record<string, unknown> }) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arnaavsareen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
