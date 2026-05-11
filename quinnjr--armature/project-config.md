---
trigger: always_on
description: This project maintains **85% code coverage** with a focus on **quality testing** over mere coverage metrics.
---


# Testing Standards

This project maintains **85% code coverage** with a focus on **quality testing** over mere coverage metrics.

## Coverage Target

### 85% Coverage Goal

```bash
# Measure coverage with cargo-tarpaulin
cargo tarpaulin --all-features --workspace --timeout 120 --out Xml --out Stdout

# Or use cargo-llvm-cov
cargo llvm-cov --all-features --workspace --html
```

**Target: 85% line coverage across the workspace**

### Quality Over Quantity

✅ **Good Coverage:**
- Tests verify actual behavior
- Tests catch real bugs
- Tests document expected behavior
- Tests are maintainable

❌ **Bad Coverage:**
- Tests just to increase percentage
- Tests that don't verify anything meaningful
- Tests that are brittle and break often
- Tests that duplicate other tests

### What Must Be Tested

**High Priority (Aim for 95%+ coverage):**
- Public API functions and methods
- Business logic and algorithms
- Error handling paths
- Edge cases and boundary conditions
- Data validation logic
- Security-critical code

**Medium Priority (Aim for 85%+ coverage):**
- Internal helper functions
- Configuration parsing
- HTTP request/response handling
- Middleware and interceptors

**Lower Priority (Aim for 60%+ coverage):**
- Simple getters/setters
- Trivial constructors
- Debug implementations
- Logging statements

**Can Skip:**
- Generated code (proc macros)
- Example code in `examples/`
- Main entry points
- Simple `#[derive]` implementations

## Testing Pyramid

### Unit Tests (70% of tests)

Test individual functions and methods in isolation.

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_user_validation_valid_email() {
        let user = User {
            email: "user@example.com".to_string(),
            name: "Alice".to_string(),
        };
        assert!(user.validate().is_ok());
    }

    #[test]
    fn test_user_validation_invalid_email() {
        let user = User {
            email: "invalid-email".to_string(),
            name: "Alice".to_string(),
        };
        let result = user.validate();
        assert!(result.is_err());
        assert!(matches!(result.unwrap_err(), ValidationError::InvalidEmail));
    }

    #[test]
    fn test_edge_case_empty_name() {
        let user = User {
            email: "user@example.com".to_string(),
            name: "".to_string(),
        };
        assert!(user.validate().is_err());
    }
}
```

### Integration Tests (25% of tests)

Test how components work together.

```rust
// tests/integration_tests.rs
use armature::prelude::*;
use armature_testing::*;

#[tokio::test]
async fn test_user_registration_flow() {
    // Arrange
    let app = TestAppBuilder::new()
        .add_module(UserModule::new())
        .build()
        .await
        .unwrap();

    let client = TestClient::new(app);

    // Act
    let response = client
        .post("/api/users/register")
        .json(&json!({
            "email": "newuser@example.com",
            "password": "SecurePass123!"
        }))
        .send()
        .await;

    // Assert
    assert_eq!(response.status(), 201);
    let body: UserDto = response.json().await;
    assert_eq!(body.email, "newuser@example.com");
}

#[tokio::test]
async fn test_authentication_and_authorization() {
    let app = TestAppBuilder::new()
        .add_module(AuthModule::new())
        .add_module(UserModule::new())
        .build()
        .await
        .unwrap();

    let client = TestClient::new(app);

    // Register user
    let register_response = client
        .post("/api/auth/register")
        .json(&register_dto)
        .send()
        .await;

    assert_eq!(register_response.status(), 201);

    // Login
    let login_response = client
        .post("/api/auth/login")
        .json(&login_dto)
        .send()
        .await;

    let token = login_response.json::<TokenResponse>().await.token;

    // Access protected route
    let protected_response = client
        .get("/api/users/me")
        .bearer_auth(&token)
        .send()
        .await;

    assert_eq!(protected_response.status(), 200);
}
```

### End-to-End Tests (5% of tests)

Test complete user workflows (fewer, more expensive tests).

```rust
// tests/e2e_tests.rs
#[tokio::test]
async fn test_complete_user_journey() {
    // Start real server
    let app = Application::create(AppModule);
    let server = tokio::spawn(async move {
        app.listen(8080).await.unwrap();
    });

    // Give server time to start
    tokio::time::sleep(Duration::from_millis(100)).await;

    let client = reqwest::Client::new();

    // Complete workflow: register -> login -> create post -> fetch post -> delete
    // ... full E2E test

    // Cleanup
    server.abort();
}
```

## Testing Best Practices

### AAA Pattern (Arrange-Act-Assert)

```rust
#[test]
fn test_calculate_discount() {
    // Arrange
    let original_price = 100.0;
    let discount_percentage = 20.0;

    // Act
    let discounted_price = calculate_discount(original_price, discount_percentage);

    // Assert
    assert_eq!(discounted_price, 80.0);
}
```

### Test One Thing Per Test

```rust
// ✅ Good: Each test verifies one specific behavior
#[test]
fn test_valid_email_passes_validation() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
