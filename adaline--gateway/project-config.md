---
trigger: always_on
description: This is a TypeScript-based AI gateway service that provides unified interfaces for multiple AI providers (OpenAI, Anthropic, Google, etc.) with features like caching, queuing, analytics, and telemetry.
---


# Adaline Gateway Repository - Main Rules

## Repository Overview

This is a TypeScript-based AI gateway service that provides unified interfaces for multiple AI providers (OpenAI, Anthropic, Google, etc.) with features like caching, queuing, analytics, and telemetry.

## Core Architecture Principles

### 1. Type Safety & Schema Validation

#### Rules

- **ALWAYS** use Zod schemas for runtime validation
- **ALWAYS** export both the schema and inferred types
- **NEVER** use `any` type - use proper generic constraints
- **ALWAYS** validate input/output at runtime using schemas

#### Instructions

```typescript
// ✅ CORRECT: Define schema first, then type
import { z } from 'zod';

const UserConfig = z.object({
  apiKey: z.string().min(1),
  model: z.string().min(1),
  temperature: z.number().min(0).max(2).default(1.0),
});
type UserConfigType = z.infer<typeof UserConfig>;

// Export both schema and type
export { UserConfig, type UserConfigType };

// ❌ INCORRECT: Using any type
function processData(data: any): any {
  return data; // Unsafe and loses type information
}

// ✅ CORRECT: Proper typing with validation
function processData(data: unknown): UserConfigType {
  return UserConfig.parse(data); // Validates at runtime
}
```

#### Implementation Steps

1. **Install Zod**: `npm install zod`
2. **Define Schema**: Create Zod schema with proper constraints
3. **Infer Type**: Use `z.infer<typeof SchemaName>` for TypeScript types
4. **Export Both**: Export schema for runtime validation, type for compile-time checking
5. **Validate Input**: Use `schema.parse()` or `schema.safeParse()` for validation

### 2. Error Handling

#### Rules

- **ALWAYS** use custom error classes extending `GatewayError`
- **ALWAYS** provide meaningful error messages with context
- **ALWAYS** handle errors gracefully with proper logging
- **NEVER** let unhandled errors bubble up

#### Instructions

```typescript
// ✅ CORRECT: Custom error class with context
import { GatewayError } from "./errors";

export class ProviderConnectionError extends GatewayError {
  constructor(
    message: string,
    public readonly provider: string,
    public readonly statusCode: number,
    public readonly originalError?: Error
  ) {
    super(`Failed to connect to ${provider}: ${message} (Status: ${statusCode})`, "PROVIDER_CONNECTION_ERROR");
  }
}

// Usage in code
try {
  await provider.makeRequest();
} catch (error) {
  if (error instanceof HttpError) {
    throw new ProviderConnectionError("API request failed", "anthropic", error.status, error);
  }
  throw error;
}
```

#### Implementation Steps

1. **Extend Base Error**: Create custom error class extending `GatewayError`
2. **Include Context**: Add relevant properties for debugging
3. **Meaningful Messages**: Provide clear, actionable error messages
4. **Error Wrapping**: Wrap external errors with custom context
5. **Proper Logging**: Log errors with full context before re-throwing

### 3. Logging & Telemetry

#### Rules

- **ALWAYS** use the centralized logger from `LoggerManager`
- **ALWAYS** include relevant context in log messages
- **ALWAYS** use OpenTelemetry for tracing and metrics
- **ALWAYS** log at appropriate levels (debug, info, warn, error)

#### Instructions

```typescript
// ✅ CORRECT: Centralized logging with context
import { LoggerManager } from "./plugins/logger";

const logger = LoggerManager.getLogger();

// Structured logging with context
logger?.info("Processing chat request", {
  requestId: request.id,
  model: request.model,
  provider: request.provider,
  timestamp: new Date().toISOString(),
});

// Error logging with full context
logger?.error("Provider request failed", {
  error: error.message,
  stack: error.stack,
  requestId: request.id,
  provider: request.provider,
  statusCode: response.status,
});

// Debug logging for troubleshooting
logger?.debug("Cache operation", {
  operation: "get",
  key: cacheKey,
  hit: !!cachedResponse,
  ttl: cachedResponse?.ttl,
});
```

#### Implementation Steps

1. **Get Logger**: Use `LoggerManager.getLogger()` to get logger instance
2. **Choose Level**: Use appropriate log level (debug, info, warn, error)
3. **Add Context**: Include relevant metadata in log messages
4. **Structured Format**: Use objects for structured logging
5. **Performance**: Use optional chaining to avoid null checks

### 4. Testing Standards

#### Rules

- **ALWAYS** write comprehensive tests for new functionality
- **ALWAYS** use Vitest as the testing framework
- **ALWAYS** mock external dependencies
- **ALWAYS** test both success and error scenarios
- **ALWAYS** use descriptive test names and proper assertions

#### Instructions

```typescript
// ✅ CORRECT: Comprehensive test structure
import { beforeEach, describe, expect, it, vi } from "vitest";

import { Gateway } from "./gateway";

describe("Gateway.completeChat", () => {
  let gateway: Gateway;
  let mockProvider: any;

  beforeEach(() => {
    // Setup mocks
    mockProvider = {
      completeChat: vi.fn(),
      getModelPricing: vi.fn(),
    };

    gateway = new Gateway({
      providers: { anthropic: mockProvider },
    });
  });


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adaline/gateway](https://github.com/adaline/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
