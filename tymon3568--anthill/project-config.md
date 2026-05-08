---
trigger: always_on
description: Multi-tenant inventory management SaaS built with Rust modular monolith and event-driven architecture.
---

# Anthill - Inventory SaaS Platform

Multi-tenant inventory management SaaS built with Rust modular monolith and event-driven architecture.

## Architecture Overview

### 3-Crate Service Pattern
Every service follows strict separation:
- **`api/`**: Axum HTTP handlers, routing, OpenAPI docs, binary entry point
- **`core/`**: Business logic traits, domain entities, DTOs - **zero infrastructure deps**
- **`infra/`**: PostgreSQL repositories, service implementations, external clients

**Dependency flow**: `api → infra → core → shared/*`

Example: `user_service_api` depends on `user_service_infra`, which depends on `user_service_core`.

### Shared Libraries
Located in `shared/`:
- `error`: `AppError` enum with `IntoResponse`, standardized error codes
- `config`: Environment config loader (`Config::from_env()`)
- `db`: `init_pool()` for PostgreSQL connection pooling
- `auth`: Casbin enforcer, JWT validation, auth extractors (`AuthUser`, `RequireAdmin`)

All services **must** use shared crates instead of duplicating code.

## Multi-Tenancy Implementation

### Database Schema Rules
Every tenant-scoped table:
1. **MUST** have `tenant_id UUID NOT NULL`
2. Use composite indexes: `(tenant_id, <other_columns>)` for performance
3. Include `tenant_id` in composite foreign keys:
   ```sql
   FOREIGN KEY (tenant_id, product_id) REFERENCES products(tenant_id, product_id)
   ```

### Application-Level Isolation
**We use application filtering, NOT Postgres RLS.**

All queries through Repository layer must include `tenant_id`:
```rust
pub async fn find_by_id(&self, ctx: &TenantContext, id: Uuid) -> Result<Product> {
    sqlx::query_as!(Product,
        "SELECT * FROM products WHERE tenant_id = $1 AND product_id = $2",
        ctx.tenant_id, id
    )
    .fetch_one(&self.pool)
    .await
}
```

Extract `tenant_id` from JWT claims in middleware, inject into request context.

## Database Standards

### Use UUID v7 (Not v4)
Timestamp-prefixed for better index locality:
```rust
use uuid::Uuid;
let id = Uuid::now_v7();
```

**Setup required:**
1. Add v7 feature to `Cargo.toml`:
   ```toml
   uuid = { version = "1.0", features = ["v7", "serde"] }
   ```
2. Compile with unstable flag (UUID v7 is currently unstable):
   ```bash
   RUSTFLAGS="--cfg uuid_unstable" cargo build
   ```

### Money as BIGINT (cents)
Never use floating-point for currency:
```rust
// Store: $10.50 → 1050 cents, 100.000 VND → 100000
pub struct Money(i64);
```

### Soft Delete Pattern
Add to critical tables:
```sql
ALTER TABLE products ADD COLUMN deleted_at TIMESTAMPTZ;
CREATE INDEX idx_products_active ON products(tenant_id, sku) WHERE deleted_at IS NULL;
```

### Timestamps
Always `TIMESTAMPTZ` with defaults:
```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
```

## Authentication & Authorization

### Email/Password Authentication
User Service handles all authentication internally:
1. **User registration** → Email/password with tenant context
2. **User login** → Validate credentials, return JWT tokens
3. **JWT issuance** → User Service generates and signs JWTs
4. **Token validation** → Services validate JWT using shared secret
5. **Session management** → Sessions stored in database

JWT claims structure:
```rust
{
  "sub": "uuid-of-user",
  "tenant_id": "uuid-of-tenant",
  "email": "user@example.com",
  "role": "admin",
  "exp": 1234567890,
  "token_type": "access"
}
```

### Auth Endpoints (User Service)
```rust
// Register new user + create/join tenant
POST /api/v1/auth/register { email, password, full_name, tenant_name }
  → Create tenant (if new) or join existing
  → Hash password with bcrypt
  → Return JWT tokens

// Login
POST /api/v1/auth/login { email, password }
  Headers: X-Tenant-ID: tenant-slug (or from subdomain)
  → Validate credentials
  → Return JWT tokens

// Refresh token
POST /api/v1/auth/refresh { refresh_token }
  → Validate refresh token
  → Return new access token

// Logout
POST /api/v1/auth/logout { refresh_token }
  → Revoke session
```

### Casbin RBAC
Multi-tenant model: `(subject, tenant, resource, action)`
- Policies stored in `casbin_rule` table
- Enforcer in `shared/auth` with PostgreSQL adapter
- Middleware: `shared_auth::casbin_middleware`
- Works with JWT: extract `user_id`, `tenant_id`, `role` to enforce policies

### Auth Extractors
From `shared/auth/extractors.rs`:
```rust
// Validate JWT and extract claims
async fn handler(user: AuthUser) -> String { 
    // user.user_id: UUID from "sub" claim
    // user.tenant_id: from JWT claim
    // user.email: from "email" claim
    // user.role: from "role" claim
}

// Admin-only endpoints
async fn admin_handler(RequireAdmin(user): RequireAdmin) -> String { ... }

// Casbin permission check
async fn protected(perm: RequirePermission) -> String { ... }
```

## Development Workflow

### Running Services
```bash
# Check entire workspace
cargo check --workspace

# Run user service (port 8000)
cargo run --bin user-service

# With auto-reload
cargo watch -x 'run --bin user-service'

# Export OpenAPI specs
cargo build --features export-spec
```

### Database Migrations
Using `sqlx-cli`:
```bash
# Run migrations
sqlx migrate run

# Create new migration
sqlx migrate add <name>

# Revert last

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tymon3568/anthill](https://github.com/tymon3568/anthill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
