---
trigger: always_on
description: **Purpose:** Design patterns, architecture, and performance optimizations for real-world applications. Use this when building web APIs, CLI tools, or any application requiring proper structure and scalability.
---

# Rust Intermediate - Additional Patterns

**Purpose:** Design patterns, architecture, and performance optimizations for real-world applications. Use this when building web APIs, CLI tools, or any application requiring proper structure and scalability.

**When to use:** Production applications, team projects, systems requiring testing/mocking, async programming, or performance-sensitive code.

## Design Patterns

**Rule: Use Builder pattern for structs with 4+ parameters or complex optional configuration.**
Why: Prevents parameter confusion and enables future extensibility without breaking changes.

```rust
pub struct DatabaseConfig {
    host: String,
    port: u16,
    database: String,
    pool_size: Option<u32>,
}

impl DatabaseConfig {
    pub fn builder() -> DatabaseConfigBuilder { DatabaseConfigBuilder::default() }
}

#[derive(Default)]
pub struct DatabaseConfigBuilder {
    host: Option<String>,
    port: Option<u16>,
    database: Option<String>,
    pool_size: Option<u32>,
}

impl DatabaseConfigBuilder {
    pub fn host(mut self, host: impl Into<String>) -> Self {
        self.host = Some(host.into()); self
    }
    
    pub fn build(self) -> Result<DatabaseConfig, BuildError> {
        Ok(DatabaseConfig {
            host: self.host.ok_or(BuildError::MissingHost)?,
            port: self.port.unwrap_or(5432),
            database: self.database.ok_or(BuildError::MissingDatabase)?,
            pool_size: self.pool_size,
        })
    }
}
```

**Rule: Use Factory pattern for creating different implementations based on runtime conditions.**
Why: Decouples object creation from usage, essential for dependency injection and testing.

```rust
pub trait UserRepository {
    fn find_by_id(&self, id: UserId) -> Result<Option<User>, RepositoryError>;
}

pub enum StorageType { InMemory, Database(String) }

impl RepositoryFactory {
    pub fn create(storage_type: StorageType) -> Result<Box<dyn UserRepository>, Error> {
        match storage_type {
            StorageType::InMemory => Ok(Box::new(InMemoryRepo::new())),
            StorageType::Database(url) => Ok(Box::new(DatabaseRepo::new(&url)?)),
        }
    }
}
```

## Module Organization

**Rule: Group related functionality into modules with clear public interfaces.**
Why: Improves maintainability and enables better encapsulation.

```rust
// lib.rs
pub mod models;
pub mod repositories;  
pub mod services;

pub use models::User;
pub use services::UserService;

// services/user_service.rs
pub struct UserService<R: UserRepository> {
    repository: R,
    validator: UserValidator,
}

impl<R: UserRepository> UserService<R> {
    pub async fn create_user(&self, request: CreateUserRequest) -> Result<User, UserServiceError> {
        self.validator.validate(&request)?;
        
        if self.repository.exists_by_email(&request.email).await? {
            return Err(UserServiceError::EmailAlreadyExists);
        }
        
        let user = User::new(request.email, request.name)?;
        self.repository.save(&user).await?;
        Ok(user)
    }
}
```

## Advanced Error Handling

**Rule: Create domain-specific error types with context and use type aliases.**
Why: Provides better debugging information and cleaner function signatures.

```rust
#[derive(Debug, thiserror::Error)]
pub enum UserServiceError {
    #[error("Validation failed: {0}")]
    Validation(#[from] ValidationError),
    #[error("User with email '{email}' already exists")]
    EmailAlreadyExists { email: String },
    #[error("Repository error: {0}")]
    Repository(#[from] RepositoryError),
}

pub type UserResult<T> = Result<T, UserServiceError>;

// Usage becomes cleaner
pub async fn find_user(&self, id: UserId) -> UserResult<User> {
    self.repository.find_by_id(id).await?
        .ok_or_else(|| UserServiceError::UserNotFound { id })
}
```

## Enhanced Testing

**Rule: Use parameterized tests and fixtures to reduce test duplication.**
Why: Provides comprehensive coverage with minimal boilerplate.

```rust
use rstest::*;

#[rstest]
#[case("test@example.com", true)]
#[case("invalid-email", false)]
#[case("", false)]
fn test_email_validation(#[case] email: &str, #[case] expected: bool) {
    assert_eq!(is_valid_email(email), expected);
}

// Test fixtures
struct TestSetup {
    service: UserService<MockUserRepository>,
}

impl TestSetup {
    fn new() -> Self {
        let mock_repo = MockUserRepository::new();
        let service = UserService::new(mock_repo);
        Self { service }
    }
}

#[tokio::test]
async fn test_create_user_success() {
    let setup = TestSetup::new();
    // Configure mocks and test...
}
```

## Async Programming

**Rule: Use async traits for I/O operations and concurrent processing for independent tasks.**
Why: Enables non-blocking operations and better scalability.

```rust
#[async_trait]
pub trait EmailService: Send + Sync {
    async fn send_email(&self, to: &str, subject: &str, body: &str) -> Result<(), EmailError>;
}

// Concurrent processing
use futures::future::join_all;

pub async fn process_users_concurrently<F, Fut>(
    users: Vec<User>,
    processor: F,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
