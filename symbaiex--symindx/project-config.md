---
trigger: always_on
description: **Rule Priority:** Core Architecture
---

# TypeScript & Bun Development Standards 2025

**Rule Priority:** Core Architecture  
**Activation:** Always Active  
**Scope:** All TypeScript/JavaScript development

## 2025 TypeScript Configuration Standards

### Ultra-Strict Type Safety
```typescript
// REQUIRED: Advanced TypeScript 5.6+ configuration
{
  "compilerOptions": {
    "strict": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "moduleDetection": "force",
    "target": "ES2025",
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

### Type Definitions 2025

- **Use `satisfies` operator** for type narrowing with inference preservation
- **Leverage `const` assertions** with template literal types
- **Implement branded types** for domain-specific validation
- **Utilize mapped types** with conditional logic for complex transformations

```typescript
// GOOD: 2025 pattern with satisfies and branded types
type UserId = string & { readonly __brand: unique symbol };
type EmailAddress = string & { readonly __brand: unique symbol };

const userConfig = {
  id: "user123" as UserId,
  email: "user@example.com" as EmailAddress,
  settings: {
    theme: "dark",
    notifications: true
  }
} satisfies UserConfig;

// GOOD: Advanced mapped type patterns
type StrictPick<T, K extends keyof T> = {
  [P in K]: T[P];
} & { [P in Exclude<keyof T, K>]?: never };

// GOOD: Template literal validation
type HTTPMethod = `${'GET' | 'POST' | 'PUT' | 'DELETE'}`;
type APIEndpoint<T extends string> = `/api/v1/${T}`;
```

## Bun Runtime Standards 2025

### Advanced Bun 1.2+ Features

```bash
# REQUIRED: Use Bun's latest performance optimizations
bun install --frozen-lockfile    # Production builds
bun add --dev --optional         # Development dependencies
bun --bun run build             # Force Bun runtime (not Node.js)
bun build --target browser --outdir dist --splitting
```

### High-Performance Patterns

- **Use `Bun.file()` with streaming** for large file operations
- **Leverage `Bun.spawn()` with IPC** for multi-process coordination
- **Implement `Bun.serve()` with WebSocket upgrades** for real-time features
- **Utilize built-in PostgreSQL driver** for database operations

```typescript
// GOOD: Bun 1.2 native PostgreSQL integration
import { Database } from 'bun:sqlite';
import postgres from 'bun:postgres'; // Native driver

// High-performance file streaming
const file = Bun.file('./large-dataset.json');
const stream = file.stream();

// WebSocket server with HTTP/2 support
export default {
  port: 3000,
  fetch(req: Request, server: Server) {
    if (server.upgrade(req)) {
      return; // WebSocket upgrade handled
    }
    return new Response("Regular HTTP response");
  },
  websocket: {
    message(ws, message) {
      ws.send(`Echo: ${message}`);
    },
  },
  error(error) {
    return new Response(`Error: ${error.message}`, { status: 500 });
  },
};
```

## 2025 Error Handling Patterns

### AI-Assisted Error Recovery
```typescript
// 2025 pattern: Self-healing error handlers with context
export abstract class SYMindXError extends Error {
  abstract readonly code: string;
  abstract readonly category: ErrorCategory;
  abstract readonly severity: 'low' | 'medium' | 'high' | 'critical';
  
  constructor(
    message: string,
    public readonly context?: Record<string, unknown>,
    public readonly recoveryHint?: string,
    public readonly aiAssisted: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
  }

  toStructuredLog() {
    return {
      error: this.name,
      code: this.code,
      category: this.category,
      severity: this.severity,
      message: this.message,
      context: this.context,
      recovery: this.recoveryHint,
      timestamp: new Date().toISOString(),
      aiEnabled: this.aiAssisted
    };
  }
}

// Enhanced Result pattern with recovery strategies
export type EnhancedResult<T, E = SYMindXError> = 
  | { success: true; data: T; metrics?: PerformanceMetrics }
  | { success: false; error: E; recovery?: () => Promise<EnhancedResult<T, E>> };
```

### Async Error Handling with Observability

```typescript
// GOOD: 2025 observability-first error handling
export async function executeWithTelemetry<T>(
  operation: string,
  fn: () => Promise<T>,
  fallback?: () => Promise<T>
): Promise<EnhancedResult<T>> {
  const startTime = performance.now();
  const traceId = crypto.randomUUID();
  
  try {
    console.time(`${operation}:${traceId}`);
    const result = await fn();
    const metrics = {
      duration: performance.now() - startTime,
      traceId,
      operation
    };
    
    return { success: true, data: result, metrics };
  } catch (error) {
    const enhancedError = new OperationError(
      `Failed to execute ${operation}`,
      { operation, traceId, duration: performance.now() - startTime },
      fallback ? "Fallback available" : "No recovery strategy",
      true
    );
    
    if (fallback) {
      return {
        success: false,
        error: enhancedError,
        recovery: () => executeWithTelemetry(`${operation}:fallback`, fallback)
      };
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
