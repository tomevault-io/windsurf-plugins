---
trigger: always_on
description: **Every phase follows TDD:**
---

# Copilot Instructions for Treasury26

## Test-Driven Development Approach

**Every phase follows TDD:**
1. Write integration test first (can fail initially)
2. Write unit tests for the component
3. Implement the minimum code to pass tests
4. Refactor while keeping tests green

Integration tests are written early and updated as new functionality is added. They serve as living documentation and ensure components work together correctly.

## Testing Guidelines

### Hard Assertions in Tests
Always use hard assertions in tests without fallbacks. Tests should fail fast with explicit error messages.

**Do:**
```rust
assert!(!page1.is_empty(), "Page 1 should not be empty");
let change = changes.first().expect("Should have at least one change");
```

**Don't:**
```rust
if !page1.is_empty() {
    // test logic
}
if let Some(change) = changes.first() {
    // test logic
}
```

This ensures tests fail immediately with clear error messages rather than silently continuing when data is missing.

### Use API Methods in Tests
When tests need to register accounts, create resources, or perform actions that have API endpoints, use the actual API routes instead of inserting directly into the database. This ensures tests validate the full request path including validation, side effects (like setting `dirty_at`, granting credits), and response handling.

**Do:**
```rust
// Register account via API endpoint (same as frontend)
let app_state = nt_be::AppState::builder()
    .db_pool(pool.clone())
    .build()
    .await?;
let app = nt_be::routes::create_routes(Arc::new(app_state));

let response = app
    .oneshot(
        axum::http::Request::builder()
            .method("POST")
            .uri("/api/monitored-accounts")
            .header("content-type", "application/json")
            .body(axum::body::Body::from(
                serde_json::json!({ "accountId": ACCOUNT_ID }).to_string(),
            ))
            .unwrap(),
    )
    .await
    .unwrap();

assert_eq!(response.status(), axum::http::StatusCode::OK);
```

**Don't:**
```rust
// Directly inserting into the database bypasses validation and side effects
sqlx::query!(
    "INSERT INTO monitored_accounts (account_id) VALUES ($1)",
    account_id
)
.execute(&pool)
.await?;
```

### Prefer End-to-End Tests Over Redundant Unit Tests
Don't create multiple tests that cover the same logic at different levels. Instead, write one test that covers the full end-to-end scenario. A monitoring cycle test that registers via API, runs the cycle, and checks results already covers `fill_gaps` — there's no need for a separate `fill_gaps` test.

### No Test Simulations
Never simulate or fake behavior to make tests pass. Tests must call the actual implementation and fail when functionality is incomplete.

**Do:**
```rust
// Test calls the actual monitoring system
run_monitor_cycle(&pool, &network, up_to_block).await?;

// Verify the system automatically discovered and tracked the token
let tokens = get_tracked_tokens(&pool, account_id).await?;
assert!(tokens.contains("discovered-token.near"));
```

**Don't:**
```rust
// Manually simulating what the system should do
let discovered = discover_tokens_manually(...);
fill_gaps(&pool, &network, account_id, "discovered-token.near", up_to_block).await?;

// Test passes but doesn't validate the real implementation
```

This ensures tests drive implementation through TDD - they fail until the real functionality is complete.

## RPC Fixture Recording

Tests that hit NEAR RPC or external APIs run through a caching proxy in CI. The proxy serves pre-recorded responses from `nt-be/tests/fixtures/rpc_cache.tar.zst`. When you add a test that makes new RPC calls, the CI will fail with a `502 Cache miss` error.

**To record new fixtures:**

```bash
cd nt-be
./scripts/record-rpc-fixtures.sh
```

This starts the proxy in RECORD mode, runs the full test suite through it, compresses the fixtures, and tells you to commit the updated archive. You can also record fixtures for a single test:

```bash
cd nt-be
# Start proxy in RECORD mode
RECORD=1 CACHE_DIR=tests/fixtures/rpc_cache PORT=18552 cargo run --bin rpc_cache_proxy &

# Run your specific test through the proxy
NEAR_RPC_URL=http://127.0.0.1:18552/near-rpc \
NEAR_ARCHIVAL_RPC_URL=http://127.0.0.1:18552/near-archival \
TRANSFER_HINTS_BASE_URL=http://127.0.0.1:18552/fastnear-hints \
NEARDATA_BASE_URL=http://127.0.0.1:18552/neardata \
INTENTS_EXPLORER_API_URL=http://127.0.0.1:18552/intents-explorer/api/v0 \
DATABASE_URL=postgresql://treasury_test:test_password@localhost:5433/treasury_test_db \
  cargo test --test your_test_name

# Kill proxy, compress, and commit
kill %1
tar -cf tests/fixtures/rpc_cache.tar -C tests/fixtures rpc_cache
zstd -f --rm -19 tests/fixtures/rpc_cache.tar -o tests/fixtures/rpc_cache.tar.zst
```

**Important:** Use the test database (`DATABASE_URL` pointing to `localhost:5433`) when recording, not the dev database. Some tests skip RPC calls when they find existing data in the database.

## Pre-Commit Checks

Always run `cargo fmt` and `cargo clippy` before committing. Code that doesn't pass formatting or has clippy warnings should not be committed.

```bash
cargo fmt
cargo clippy --all-targets
```

## Pull Request Guidelines

### Conventional Commits


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NEAR-DevHub/trezu](https://github.com/NEAR-DevHub/trezu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
