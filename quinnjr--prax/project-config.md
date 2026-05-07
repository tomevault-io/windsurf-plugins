---
trigger: always_on
description: This project provides a tiered caching layer with in-memory and Redis backends. Follow these guidelines for effective cache usage.
---

# Data Caching Guidelines

This project provides a tiered caching layer with in-memory and Redis backends. Follow these guidelines for effective cache usage.

## Cache Architecture

### Tiered Cache (Recommended)

Use L1 (memory) + L2 (Redis) for best performance:

```rust
use prax_query::data_cache::{TieredCache, MemoryCache, RedisCache};

// L1: Fast in-memory cache
let memory = MemoryCache::builder()
    .max_capacity(10_000)
    .time_to_live(Duration::from_secs(60))
    .build();

// L2: Distributed Redis cache
let redis = RedisCache::builder()
    .url("redis://localhost:6379")
    .key_prefix("myapp:")
    .default_ttl(Duration::from_secs(300))
    .build()
    .await?;

// Tiered: Check L1 first, then L2
let cache = TieredCache::new(memory, redis);
```

### Memory-Only Cache

For single-instance deployments:

```rust
use prax_query::data_cache::MemoryCache;

let cache = MemoryCache::builder()
    .max_capacity(50_000)
    .time_to_live(Duration::from_secs(300))
    .time_to_idle(Duration::from_secs(60))
    .build();
```

### Redis-Only Cache

For distributed caching without local cache:

```rust
use prax_query::data_cache::RedisCache;

let cache = RedisCache::builder()
    .url("redis://localhost:6379")
    .pool_size(10)
    .key_prefix("myapp:")
    .default_ttl(Duration::from_secs(3600))
    .build()
    .await?;
```

## Cache Keys

### Use Structured Keys

```rust
use prax_query::data_cache::CacheKey;

// ✅ Good: Structured, predictable keys
let key = CacheKey::entity("User", 123);           // "User:123"
let key = CacheKey::query("users", &filter_hash);  // "query:users:{hash}"
let key = CacheKey::custom("feature_flags", "v1"); // "feature_flags:v1"

// ❌ Bad: Unstructured keys
let key = format!("user_{}", id);  // No namespace, hard to invalidate
```

### Include Tenant in Keys (Multi-Tenant)

```rust
// ✅ Good: Tenant-scoped keys
let key = CacheKey::tenant_entity(tenant_id, "User", user_id);
// "tenant:123:User:456"

// ✅ Good: Tenant prefix in Redis
let redis = RedisCache::builder()
    .key_prefix(format!("tenant:{}:", tenant_id))
    .build()
    .await?;

// ❌ DANGEROUS: Shared keys across tenants
let key = CacheKey::entity("User", user_id);
// Tenant A might see Tenant B's cached data!
```

## Cache Operations

### Basic Get/Set

```rust
// Get with type inference
let user: Option<User> = cache.get(&key).await?;

// Set with default TTL
cache.set(&key, &user).await?;

// Set with custom TTL
cache.set_with_ttl(&key, &user, Duration::from_secs(600)).await?;

// Get or compute
let user = cache.get_or_set(&key, || async {
    db.user().find_unique(user::id::equals(id)).exec().await
}).await?;
```

### Batch Operations

```rust
// Get multiple keys
let keys = vec![
    CacheKey::entity("User", 1),
    CacheKey::entity("User", 2),
    CacheKey::entity("User", 3),
];
let users: Vec<Option<User>> = cache.get_many(&keys).await?;

// Set multiple
cache.set_many(&[(key1, user1), (key2, user2)]).await?;
```

## Invalidation Strategies

### Entity-Based Invalidation

```rust
// Invalidate single entity
cache.invalidate(&CacheKey::entity("User", user_id)).await?;

// Invalidate all entities of a type
cache.invalidate_pattern("User:*").await?;

// Invalidate related entities
async fn update_user(id: i64, data: UpdateUser) -> Result<User> {
    let user = db.user().update(id, data).exec().await?;

    // Invalidate user cache
    cache.invalidate(&CacheKey::entity("User", id)).await?;

    // Invalidate related caches
    cache.invalidate(&CacheKey::entity("UserProfile", id)).await?;
    cache.invalidate_pattern(&format!("query:users:*")).await?;

    Ok(user)
}
```

### Tag-Based Invalidation

```rust
use prax_query::data_cache::EntityTag;

// Cache with tags
cache.set_with_tags(
    &key,
    &user,
    &[EntityTag::entity("User"), EntityTag::record("User", user_id)],
).await?;

// Invalidate by tag
cache.invalidate_tag(&EntityTag::entity("User")).await?;
// All User caches invalidated
```

### Write-Through Pattern

```rust
// Update database and cache atomically
async fn update_user(id: i64, data: UpdateUser) -> Result<User> {
    // Update DB
    let user = db.user().update(id, data).exec().await?;

    // Update cache (not invalidate)
    cache.set(&CacheKey::entity("User", id), &user).await?;

    Ok(user)
}
```

### Cache-Aside Pattern

```rust
// Read: Check cache first
async fn get_user(id: i64) -> Result<User> {
    let key = CacheKey::entity("User", id);

    // Try cache
    if let Some(user) = cache.get(&key).await? {
        return Ok(user);
    }

    // Cache miss: load from DB
    let user = db.user()
        .find_unique(user::id::equals(id))
        .exec()
        .await?
        .ok_or(Error::NotFound)?;

    // Populate cache
    cache.set(&key, &user).await?;

    Ok(user)
}
```

## TTL Configuration

### Choose Appropriate TTLs

```rust
// ✅ Good: Different TTLs for different data types

// Rarely changes, long TTL
let feature_flags = CachePolicy::new()
    .ttl(Duration::from_secs(3600))  // 1 hour
    .stale_while_revalidate(Duration::from_secs(300));

// User data, medium TTL
let user_data = CachePolicy::new()
    .ttl(Duration::from_secs(300))  // 5 minutes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
