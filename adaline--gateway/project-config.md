---
trigger: always_on
description: The gateway core module is the central orchestrator that handles AI provider requests, manages caching, queuing, and provides unified interfaces for chat completion, embeddings, and tool responses.
---


# Gateway Core Module - Complete Rules & Implementation Guide

## Module Overview

The gateway core module is the central orchestrator that handles AI provider requests, manages caching, queuing, and provides unified interfaces for chat completion, embeddings, and tool responses.

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

  it("should successfully complete chat request", async () => {
    // Arrange
    const request = {
      messages: [{ role: "user", content: "Hello" }],
      model: "claude-3-sonnet",
    };

    const expectedResponse = {
      messages: [{ role: "assistant", content: "Hi there!" }],
      usage: { promptTokens: 5, completionTokens: 3, totalTokens: 8 },
    };

    mockProvider.completeChat.mockResolvedValue(expectedResponse);

    // Act
    const result = await gateway.completeChat(request);

    // Assert
    expect(result).toEqual(expectedResponse);
    expect(mockProvider.completeChat).toHaveBeenCalledWith(request);
  });

  it("should handle provider errors gracefully", async () => {
    // Arrange
    const request = { messages: [], model: "invalid-model" };
    const error = new Error("Model not found");
    mockProvider.completeChat.mockRejectedValue(error);

    // Act & Assert
    await expect(gateway.completeChat(request)).rejects.toThrow("Model not found");
  });
});
```

## Gateway Class Rules

### 1. Constructor & Initialization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adaline/gateway](https://github.com/adaline/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
