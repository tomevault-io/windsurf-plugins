---
trigger: always_on
description: TDD workflow and testing conventions for Auth9 project
---


# Testing Conventions

## TDD Development Workflow

When implementing features or fixing bugs, follow the Test-Driven Development (TDD) cycle:

```
1. RED    → Write a failing test first
2. GREEN  → Write minimal code to pass the test
3. REFACTOR → Improve code while keeping tests green
```

### TDD Rules

- **Write tests BEFORE implementation code**
- **One test at a time**: Don't write multiple failing tests
- **Minimal implementation**: Only write enough code to pass the current test
- **Refactor continuously**: Clean up after each green phase

### Coverage Requirement

**Minimum test coverage: 90%**

```bash
# Rust - check coverage with llvm-cov (not tarpaulin)
cargo llvm-cov --html

# TypeScript - check coverage with vitest
npm run test -- --coverage
```

Coverage targets by layer:
| Layer | Target |
|-------|--------|
| Domain/Business logic | 95%+ |
| Service layer | 90%+ |
| API handlers | 80%+ |
| gRPC handlers | 85%+ |

## Test Pyramid

```
        E2E Tests (few, critical flows)
              /\
             /  \
        Unit Tests (service layer with mocks)
           /      \
          /        \
     gRPC/Identity Engine Tests (wiremock, NoOpCache)
```

## auth9-core (Rust)

### Testing Strategy - NO EXTERNAL DEPENDENCIES

All tests run fast (~1-2 seconds) with **no Docker or external services**:

| Component | Testing Approach |
|-----------|-----------------|
| Repository layer | Mock traits with `mockall` |
| Service layer | Unit tests with mock repositories |
| gRPC services | `NoOpCacheManager` + mock repositories |
| Identity Engine | `wiremock` HTTP mocking |

### Prohibited

- **No testcontainers** - tests must not start Docker containers
- **No real database connections** - use mock repositories
- **No real Redis connections** - use `NoOpCacheManager`
- **No faker library** - construct test data directly

### Unit Tests (Service Layer)

```rust
// Location: src/service/*.rs inside #[cfg(test)] module
// Use mockall for mocking repository dependencies

#[cfg(test)]
mod tests {
    use super::*;
    use crate::repository::tenant::MockTenantRepository;
    use mockall::predicate::*;

    #[tokio::test]
    async fn test_create_tenant_success() {
        let mut mock = MockTenantRepository::new();

        mock.expect_find_by_slug()
            .with(eq("test-tenant"))
            .returning(|_| Ok(None));

        mock.expect_create()
            .returning(|input| Ok(Tenant {
                name: input.name.clone(),
                ..Default::default()
            }));

        let service = TenantService::new(Arc::new(mock), None);
        let result = service.create(input).await;
        assert!(result.is_ok());
    }
}
```

### gRPC Tests

```rust
// Location: tests/grpc_*.rs
// Use NoOpCacheManager instead of real Redis

use auth9_core::cache::NoOpCacheManager;

fn create_test_cache() -> NoOpCacheManager {
    NoOpCacheManager::new()
}

#[tokio::test]
async fn test_token_exchange() {
    let cache = create_test_cache();
    let service = TokenExchangeService::new(
        jwt_manager,
        cache,  // No Redis connection needed
        user_repo,
        service_repo,
        rbac_repo,
    );
    // ...
}
```

### Identity Engine Tests

```rust
// Use wiremock for HTTP mocking of identity backend calls

use wiremock::{Mock, ResponseTemplate, matchers::{method, path}};

#[tokio::test]
async fn test_identity_engine_create_user() {
    let mock_server = MockServer::start().await;

    Mock::given(method("POST"))
        .and(path("/api/users"))
        .respond_with(ResponseTemplate::new(201))
        .mount(&mock_server)
        .await;

    let engine = create_test_engine(&mock_server.uri());
    let result = engine.create_user(&input).await;
    assert!(result.is_ok());
}
```

### HTTP Handler Tests (API Layer)

HTTP handler tests use **Dependency Injection via `HasServices` trait** to test production code with mock repositories.

**Architecture:**
```
Production:  AppState (impl HasServices) → build_router() → Handlers<AppState>
Tests:       TestAppState (impl HasServices) → build_router() → Handlers<TestAppState>
                                                    ↑
                                          Same production handlers!
```

**Key Files:**
| File | Purpose |
|------|---------|
| `src/state.rs` | `HasServices` trait definition |
| `src/server/mod.rs` | `AppState` + `build_router<S: HasServices>()` |
| `tests/api/http/mod.rs` | `TestAppState` + test helpers |
| `tests/api/http/*_http_test.rs` | HTTP handler tests |

**Handler Pattern - Always use generics:**
```rust
// src/api/tenant.rs - CORRECT
pub async fn list<S: HasServices>(
    State(state): State<S>,
    Query(pagination): Query<PaginationQuery>,
) -> Result<impl IntoResponse> {
    let (tenants, total) = state
        .tenant_service()  // Method from HasServices trait
        .list(pagination.page, pagination.per_page)
        .await?;
    Ok(Json(PaginatedResponse::new(tenants, pagination.page, pagination.per_page, total)))
}
```

**Test Example:**
```rust
// tests/api/http/tenant_http_test.rs
use crate::api::http::{build_test_router, get_json, TestAppState};

#[tokio::test]
async fn test_list_tenants_returns_200() {
    let state = TestAppState::new();
    state.tenant_repo.seed_tenant(input).await;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c9r-io/auth9](https://github.com/c9r-io/auth9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
