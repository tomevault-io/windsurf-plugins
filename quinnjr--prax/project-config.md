---
trigger: always_on
description: This project provides multi-tenancy support with multiple isolation strategies. Follow these guidelines for secure and performant tenant isolation.
---

# Multi-Tenancy Guidelines

This project provides multi-tenancy support with multiple isolation strategies. Follow these guidelines for secure and performant tenant isolation.

## Isolation Strategies

### Row-Level Security (RLS) - Recommended

Most efficient for shared-table tenancy:

```rust
use prax_query::tenant::{RlsManager, RlsConfig};

// Configure RLS
let rls = RlsManager::new(
    RlsConfig::new("tenant_id")
        .with_session_variable("app.current_tenant")
        .add_tables(["users", "orders", "products"])
);

// Apply to connection
rls.set_tenant(&conn, tenant_id).await?;

// All queries automatically filtered by tenant_id
let users = client.user().find_many().exec().await?;
// SQL: SELECT * FROM users WHERE tenant_id = current_setting('app.current_tenant')
```

### Schema-Based Isolation

For stronger isolation with separate schemas:

```rust
use prax_query::tenant::{TenantPoolManager, SchemaIsolation};

let manager = TenantPoolManager::new(base_pool)
    .with_isolation(SchemaIsolation::Schema);

// Get tenant-specific pool
let pool = manager.get_pool(tenant_id).await?;
// Queries run against schema_{tenant_id}.users
```

### Database-Based Isolation

Maximum isolation with separate databases:

```rust
let manager = TenantPoolManager::new(base_pool)
    .with_isolation(SchemaIsolation::Database);

// Each tenant has own database
let pool = manager.get_pool(tenant_id).await?;
// Connects to tenant_{tenant_id} database
```

## Context Propagation

### Use Task-Local Context (Zero Allocation)

```rust
use prax_query::tenant::task_local::with_tenant;

// Set tenant context for async block
with_tenant("tenant-123", async {
    // All queries in this block use tenant-123
    let users = client.user().find_many().exec().await?;
    let orders = client.order().find_many().exec().await?;
    Ok(())
}).await?;

// ❌ Bad: Manual tenant filter on each query
let users = client.user()
    .find_many()
    .where_(user::tenant_id::equals(tenant_id)) // Easy to forget!
    .exec()
    .await?;
```

### Thread-Local for Sync Code

```rust
use prax_query::tenant::thread_local::{set_tenant, get_tenant};

// In request handler
set_tenant(tenant_id);

// Deep in call stack
let current = get_tenant().expect("tenant not set");
```

## Security Rules

### Never Trust Client-Provided Tenant ID

```rust
// ✅ Good: Extract tenant from authenticated session
async fn handler(session: Session, req: Request) -> Response {
    let tenant_id = session.tenant_id(); // From verified JWT/session

    with_tenant(tenant_id, async {
        // Process request
    }).await
}

// ❌ DANGEROUS: Accept tenant from request
async fn handler(req: Request) -> Response {
    let tenant_id = req.header("X-Tenant-ID"); // Attacker can set this!
    // ...
}
```

### Validate Cross-Tenant References

```rust
// ✅ Good: Validate foreign key belongs to same tenant
async fn create_order(tenant_id: TenantId, user_id: UserId) -> Result<Order> {
    // Verify user belongs to tenant
    let user = client.user()
        .find_unique(user::id::equals(user_id))
        .exec()
        .await?
        .ok_or(Error::NotFound)?;

    if user.tenant_id != tenant_id {
        return Err(Error::CrossTenantAccess);
    }

    // Proceed with order creation
}
```

### Enable RLS at Database Level

```sql
-- PostgreSQL RLS setup
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON users
    USING (tenant_id = current_setting('app.current_tenant')::int);

-- Force RLS even for table owners
ALTER TABLE users FORCE ROW LEVEL SECURITY;
```

## Performance Patterns

### Use Statement Caching

```rust
use prax_query::tenant::cache::StatementCache;

// Global mode: Share prepared statements across tenants (with RLS)
let cache = StatementCache::global();

// Per-tenant mode: Separate caches for schema-based isolation
let cache = StatementCache::per_tenant(1000); // LRU size per tenant
```

### Use Tenant Cache with TTL

```rust
use prax_query::tenant::cache::ShardedTenantCache;

// High-concurrency tenant cache
let cache = ShardedTenantCache::high_concurrency(10_000);

// With TTL
cache.insert(tenant_id, config, Duration::from_secs(300));

// Get or load
let config = cache.get_or_insert(tenant_id, || {
    load_tenant_config(tenant_id)
}).await?;
```

### Warm Up Tenant Pools

```rust
// Pre-warm pools for known active tenants
let active_tenants = get_active_tenant_ids().await?;
manager.warmup(&active_tenants).await?;

// Lazy pools for less active tenants
// Created on first access, evicted after idle timeout
```

## Testing Multi-Tenancy

### Test Tenant Isolation

```rust
#[tokio::test]
async fn test_tenant_isolation() {
    let tenant_a = create_test_tenant().await;
    let tenant_b = create_test_tenant().await;

    // Create data for tenant A
    with_tenant(tenant_a.id, async {
        client.user().create(user::create! { name: "Alice" }).exec().await?;
        Ok::<_, Error>(())
    }).await?;

    // Verify tenant B cannot see tenant A's data
    with_tenant(tenant_b.id, async {
        let users = client.user().find_many().exec().await?;
        assert!(users.is_empty(), "Tenant B should not see Tenant A's users");

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
