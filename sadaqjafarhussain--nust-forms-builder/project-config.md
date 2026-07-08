---
trigger: always_on
description: Caching rules for performance improvements
---

# Cache Optimization Patterns for Formbricks

## Cache Strategy Overview

Formbricks uses a **hybrid caching approach** optimized for enterprise scale:

- **Redis** for persistent cross-request caching  
- **React `cache()`** for request-level deduplication
- **NO Next.js `unstable_cache()`** - avoid for reliability

## Key Files

### Core Cache Infrastructure
- [packages/cache/src/service.ts](mdc:packages/cache/src/service.ts) - Redis cache service
- [packages/cache/src/client.ts](mdc:packages/cache/src/client.ts) - Cache client initialization and singleton management  
- [apps/web/lib/cache/index.ts](mdc:apps/web/lib/cache/index.ts) - Cache service proxy for web app
- [packages/cache/src/index.ts](mdc:packages/cache/src/index.ts) - Cache package exports and utilities

### Environment State Caching (Critical Endpoint)
- [apps/web/app/api/v1/client/[environmentId]/environment/route.ts](mdc:apps/web/app/api/v1/client/[environmentId]/environment/route.ts) - Main endpoint serving hundreds of thousands of SDK clients
- [apps/web/app/api/v1/client/[environmentId]/environment/lib/data.ts](mdc:apps/web/app/api/v1/client/[environmentId]/environment/lib/data.ts) - Optimized data layer with caching

## Enterprise-Grade Cache Key Patterns

**Always use** the `createCacheKey` utilities from the cache package:

```typescript
// ✅ Correct patterns
createCacheKey.environment.state(environmentId)     // "fb:env:abc123:state"
createCacheKey.organization.billing(organizationId) // "fb:org:xyz789:billing"
createCacheKey.license.status(organizationId)       // "fb:license:org123:status"
createCacheKey.user.permissions(userId, orgId)      // "fb:user:456:org:123:permissions"

// ❌ Never use flat keys - collision-prone
"environment_abc123"
"user_data_456"
```

## When to Use Each Cache Type

### Use React `cache()` for Request Deduplication
```typescript
// ✅ Prevents multiple calls within same request
export const getEnterpriseLicense = reactCache(async () => {
  // Complex license validation logic
});
```

### Use `cache.withCache()` for Simple Database Queries
```typescript
// ✅ Simple caching with automatic fallback (TTL in milliseconds)
export const getActionClasses = (environmentId: string) => {
  return cache.withCache(() => fetchActionClassesFromDB(environmentId), 
    createCacheKey.environment.actionClasses(environmentId),
    60 * 30 * 1000 // 30 minutes in milliseconds
  );
};
```

### Use Explicit Redis Cache for Complex Business Logic
```typescript
// ✅ Full control for high-stakes endpoints
export const getEnvironmentState = async (environmentId: string) => {
  const cached = await environmentStateCache.getEnvironmentState(environmentId);
  if (cached) return cached;
  
  const fresh = await buildComplexState(environmentId);
  await environmentStateCache.setEnvironmentState(environmentId, fresh);
  return fresh;
};
```

## Caching Decision Framework

### When TO Add Caching

```typescript
// ✅ Expensive operations that benefit from caching
- Database queries (>10ms typical)
- External API calls (>50ms typical)  
- Complex computations (>5ms)
- File system operations
- Heavy data transformations

// Example: Database query with complex joins (TTL in milliseconds)
export const getEnvironmentWithDetails = withCache(
  async (environmentId: string) => {
    return prisma.environment.findUnique({
      where: { id: environmentId },
      include: { /* complex joins */ }
    });
  },
  { key: createCacheKey.environment.details(environmentId), ttl: 60 * 30 * 1000 } // 30 minutes
)();
```

### When NOT to Add Caching

```typescript
// ❌ Don't cache these operations - minimal overhead
- Simple property access (<0.1ms)
- Basic transformations (<1ms)
- Functions that just call already-cached functions
- Pure computation without I/O

// ❌ Bad example: Redundant caching
const getCachedLicenseFeatures = withCache(
  async () => {
    const license = await getEnterpriseLicense(); // Already cached!
    return license.active ? license.features : null; // Just property access
  },
  { key: "license-features", ttl: 1800 * 1000 } // 30 minutes in milliseconds
);

// ✅ Good example: Simple and efficient
const getLicenseFeatures = async () => {
  const license = await getEnterpriseLicense(); // Already cached
  return license.active ? license.features : null; // 0.1ms overhead
};
```

### Computational Overhead Analysis

Before adding caching, analyze the overhead:

```typescript
// ✅ High overhead - CACHE IT
- Database queries: ~10-100ms
- External APIs: ~50-500ms  
- File I/O: ~5-50ms
- Complex algorithms: >5ms

// ❌ Low overhead - DON'T CACHE
- Property access: ~0.001ms
- Simple lookups: ~0.1ms
- Basic validation: ~1ms
- Type checks: ~0.01ms

// Example decision tree:
const expensiveOperation = async () => {
  return prisma.query(); // 50ms - CACHE IT
};

const cheapOperation = (data: any) => {
  return data.property; // 0.001ms - DON'T CACHE
};
```

### Avoid Cache Wrapper Anti-Pattern

```typescript
// ❌ Don't create wrapper functions just for caching
const getCachedUserPermissions = withCache(
  async (userId: string) => getUserPermissions(userId),
  { key: createCacheKey.user.permissions(userId), ttl: 3600 * 1000 } // 1 hour in milliseconds
);

// ✅ Add caching directly to the original function

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SadaqJafarHussain/NUST-Forms-Builder](https://github.com/SadaqJafarHussain/NUST-Forms-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
