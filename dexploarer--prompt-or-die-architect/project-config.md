---
trigger: always_on
description: Development best practices guide for working with ElizaOS core package
---


# ElizaOS Development Best Practices

## Core Development Principles

### Type Safety First

```typescript
// ✅ DO: Use strict TypeScript types throughout
import type {
  IAgentRuntime,
  Memory,
  Character,
  Action,
  Provider,
  State
} from '@elizaos/core';

// Avoid any types
function processMemory(memory: Memory): string {
  // TypeScript knows memory.content.text exists
  return memory.content.text || 'No content';
}

// ❌ DON'T: Use any or generic types when specific types exist
function badProcessMemory(memory: any): any {
  return memory.content?.text; // No type safety
}
```

### Error Handling Patterns

```typescript
// ✅ DO: Use structured error handling
export class ServiceError extends Error {
  constructor(
    message: string,
    public code: string,
    public context?: Record<string, any>
  ) {
    super(message);
    this.name = 'ServiceError';
  }
}

async function safeOperation(runtime: IAgentRuntime): Promise<void> {
  try {
    await runtime.createMemory({
      entityId: runtime.agentId,
      roomId: 'system-room',
      content: { text: 'Operation started' }
    });
  } catch (error) {
    if (error instanceof ServiceError) {
      await runtime.logger.error('Service operation failed', {
        code: error.code,
        context: error.context
      });
    }
    throw error;
  }
}

// ✅ DO: Use error boundaries for async operations
async function withErrorBoundary<T>(
  operation: () => Promise<T>,
  fallback: T
): Promise<T> {
  try {
    return await operation();
  } catch (error) {
    console.error('Operation failed, using fallback:', error);
    return fallback;
  }
}
```

### Resource Management

```typescript
// ✅ DO: Properly manage resources with cleanup
class ResourceManager {
  private resources = new Map<string, any>();

  async acquire<T>(key: string, factory: () => Promise<T>): Promise<T> {
    if (this.resources.has(key)) {
      return this.resources.get(key);
    }

    const resource = await factory();
    this.resources.set(key, resource);
    return resource;
  }

  release(key: string): void {
    const resource = this.resources.get(key);
    if (resource && typeof resource.close === 'function') {
      resource.close();
    }
    this.resources.delete(key);
  }

  async cleanup(): Promise<void> {
    for (const key of this.resources.keys()) {
      this.release(key);
    }
  }
}

// ✅ DO: Use resource managers in services
class DatabaseService extends Service {
  private resourceManager = new ResourceManager();

  async query(sql: string): Promise<any[]> {
    const connection = await this.resourceManager.acquire(
      'db-connection',
      () => createDatabaseConnection()
    );

    try {
      return await connection.query(sql);
    } finally {
      // Connection pooling handles cleanup automatically
    }
  }

  async stop(): Promise<void> {
    await this.resourceManager.cleanup();
  }
}
```

## Memory Management

### Efficient Memory Operations

```typescript
// ✅ DO: Use batch operations for multiple memories
async function createMultipleMemories(
  runtime: IAgentRuntime,
  memories: Omit<Memory, 'id' | 'createdAt'>[]
): Promise<UUID[]> {
  const memoryIds: UUID[] = [];

  // Use Promise.all for concurrent operations
  const results = await Promise.all(
    memories.map(memory =>
      runtime.createMemory({
        ...memory,
        createdAt: Date.now()
      })
    )
  );

  return results;
}

// ✅ DO: Implement memory pagination
async function getMemoriesPaginated(
  runtime: IAgentRuntime,
  roomId: UUID,
  page: number = 1,
  limit: number = 50
): Promise<{ memories: Memory[]; hasMore: boolean }> {
  const offset = (page - 1) * limit;

  const memories = await runtime.getMemories({
    roomId,
    count: limit + 1, // Get one extra to check if there are more
    // Add any additional filters
  });

  const hasMore = memories.length > limit;
  const pageMemories = hasMore ? memories.slice(0, limit) : memories;

  return { memories: pageMemories, hasMore };
}
```

### Memory Search Optimization

```typescript
// ✅ DO: Use efficient search patterns
async function searchWithFallback(
  runtime: IAgentRuntime,
  query: string,
  options: {
    roomId?: UUID;
    threshold?: number;
    limit?: number;
  }
): Promise<Memory[]> {
  const {
    roomId,
    threshold = 0.8,
    limit = 10
  } = options;

  // Try semantic search first
  const semanticResults = await runtime.searchMemories({
    query,
    roomId,
    match_threshold: threshold,
    count: limit
  });

  if (semanticResults.length > 0) {
    return semanticResults;
  }

  // Fallback to text-based search
  return await runtime.getMemories({
    roomId,
    count: limit,
    // Filter by text content containing query terms
  });
}
```

## State Management

### State Composition Best Practices

```typescript
// ✅ DO: Create focused state composition
async function composeFocusedState(
  runtime: IAgentRuntime,
  message: Memory,
  requiredProviders: string[]
): Promise<State> {
  // Only include necessary providers
  const state = await runtime.composeState(
    message,
    requiredProviders,
    true, // onlyInclude
    false // skipCache for fresh data
  );

  // Add computed properties
  state.values.messageLength = message.content.text?.length || 0;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dexploarer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
