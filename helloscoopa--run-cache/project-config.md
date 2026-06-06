---
trigger: always_on
description: **RunCache** is a dependency-free, lightweight runtime caching library for JavaScript and TypeScript applications. It provides in-memory caching with TTL support, automatic value regeneration, and advanced features like tag-based invalidation, dependency tracking, and persistent storage.
---

# CLAUDE.md - RunCache Project Guide

## Project Overview
**RunCache** is a dependency-free, lightweight runtime caching library for JavaScript and TypeScript applications. It provides in-memory caching with TTL support, automatic value regeneration, and advanced features like tag-based invalidation, dependency tracking, and persistent storage.

## Key Features
- Zero dependencies
- In-memory performance with TTL support
- Source function support for automatic refetching
- Event system for cache lifecycle monitoring
- Pattern matching with wildcards
- Multiple eviction policies (LRU, LFU)
- Middleware support for custom processing
- Tag-based and dependency-based invalidation
- Persistent storage adapters (localStorage, IndexedDB, filesystem)
- TypeScript support with full type definitions

## Project Structure

### Main Files
- `src/run-cache.ts` - Main RunCache class with static methods
- `src/index.ts` - Entry point and exports
- `src/core/cache-store.ts` - Core cache implementation
- `src/types/` - TypeScript type definitions
- `src/storage/` - Storage adapter implementations
- `src/logging/` - Logger implementation
- `src/policies/` - Eviction policy implementations

### Build & Testing
- **Build**: `npm run build` (TypeScript compilation + minification)
- **Test**: `npm test` (Jest)
- **Lint**: `npm run lint` (ESLint with TypeScript)
- **Coverage**: `npm run test:coverage`

### Development Commands
```bash
# Install dependencies
npm install

# Run tests
npm test
npm run test:watch    # Watch mode
npm run test:coverage # With coverage

# Build
npm run build         # Full build
npm run build:min     # Minified build only

# Linting
npm run lint          # Check for issues
npm run lint:fix      # Auto-fix issues
```

## Architecture

### Core Components
1. **RunCache Class** (`src/run-cache.ts`): Static wrapper providing public API
2. **CacheStore** (`src/core/cache-store.ts`): Core cache implementation
3. **Event System** (`src/core/event-system.ts`): Event handling and subscriptions
4. **Middleware Manager** (`src/core/middleware-manager.ts`): Middleware pipeline
5. **Storage Adapters** (`src/storage/`): Persistence layer implementations

### Key Patterns
- **Singleton Pattern**: RunCache uses a single static instance
- **Event-Driven**: Comprehensive event system for monitoring
- **Middleware Pattern**: Pluggable transformation pipeline
- **Strategy Pattern**: Multiple eviction policies and storage adapters

## API Reference

### Basic Operations
```typescript
// Set cache entry
await RunCache.set({ 
  key: "user:123", 
  value: "John Doe", 
  ttl: 60000,  // 1 minute
  tags: ["user", "profile"],
  dependencies: ["user:session"]
});

// Get cache entry
const user = await RunCache.get("user:123");

// Delete entry
await RunCache.delete("user:123");

// Check existence
const exists = await RunCache.has("user:*");

// Refetch (requires sourceFn)
await RunCache.refetch("user:123");
```

### Advanced Features
```typescript
// Auto-refetch configuration
await RunCache.set({
  key: "api-data",
  sourceFn: async () => await fetchApiData(),
  ttl: 300000,
  autoRefetch: true
});

// Tag-based invalidation
RunCache.invalidateByTag("user");

// Dependency-based invalidation
RunCache.invalidateByDependency("user:session");

// Middleware
RunCache.use(async (value, context, next) => {
  console.log(`${context.operation} on ${context.key}`);
  return next(value);
});
```

### Configuration
```typescript
import { EvictionPolicy, LocalStorageAdapter } from 'run-cache';

RunCache.configure({
  maxEntries: 1000,
  evictionPolicy: EvictionPolicy.LRU,
  debug: true,
  storageAdapter: new LocalStorageAdapter()
});
```

### Event Handling
```typescript
// Global events
RunCache.onExpiry((event) => console.log(`Expired: ${event.key}`));
RunCache.onRefetch((event) => console.log(`Refetched: ${event.key}`));
RunCache.onRefetchFailure((event) => console.error(`Failed: ${event.key}`));

// Key-specific events (supports wildcards)
RunCache.onKeyExpiry("user:*", (event) => {
  console.log(`User cache expired: ${event.key}`);
});
```

## Common Development Tasks

### Adding New Features
1. Update type definitions in `src/types/`
2. Implement core logic in `src/core/cache-store.ts`
3. Add static wrapper methods in `src/run-cache.ts`
4. Add comprehensive tests in `.test.ts` files
5. Update documentation

### Testing Strategy
- Unit tests for each component (`.unit.test.ts`)
- Integration tests for main functionality (`.test.ts`)
- Coverage target: Keep above 90%
- Test files use Jest with TypeScript support

### Storage Adapters
Located in `src/storage/`:
- `local-storage-adapter.ts` - Browser localStorage
- `indexed-db-adapter.ts` - Browser IndexedDB  
- `filesystem-adapter.ts` - Node.js filesystem

Creating new adapters:
1. Implement `StorageAdapter` interface
2. Add to `src/storage/index.ts`
3. Create comprehensive tests
4. Update documentation

## Debugging

### Debug Mode
```typescript
RunCache.configure({ debug: true });
```

### Common Issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helloscoopa/run-cache](https://github.com/helloscoopa/run-cache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
