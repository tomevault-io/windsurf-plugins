---
trigger: always_on
description: Enterprise-grade patterns for production systems including security, complex architecture, and deployment. Use this for high-stakes applications requiring authentication, comprehensive testing, and production deployment.
---

# Rust Advanced - Complex Scenarios

**Purpose:** Enterprise-grade patterns for production systems including security, complex architecture, and deployment. Use this for high-stakes applications requiring authentication, comprehensive testing, and production deployment.

**When to use:** Production services, security-critical applications, enterprise systems, microservices, or any system requiring bulletproof reliability and performance.

## Architecture Patterns

**Rule: Use dependency injection with traits for testable, decoupled business logic.**
Why: Enables testing with mocks, follows SOLID principles, and allows swapping implementations.

```rust
#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn find_by_id(&self, id: UserId) -> RepositoryResult<Option<User>>;
    async fn save(&self, user: &User) -> RepositoryResult<()>;
}

#[async_trait]
pub trait EventPublisher: Send + Sync {
    async fn publish(&self, event: DomainEvent) -> Result<(), EventError>;
}

pub struct UserDomainService<R, E> 
where R: UserRepository, E: EventPublisher {
    repository: Arc<R>,
    event_publisher: Arc<E>,
}

impl<R, E> UserDomainService<R, E>
where R: UserRepository, E: EventPublisher {
    pub async fn create_user(&self, cmd: CreateUserCommand) -> DomainResult<User> {
        let user = User::create(cmd.email, cmd.name, cmd.role)?;
        self.repository.save(&user).await?;
        
        let event = DomainEvent::UserCreated {
            user_id: user.id(),
            email: user.email().to_string(),
            created_at: Utc::now(),
        };
        self.event_publisher.publish(event).await?;
        Ok(user)
    }
}
```

**Rule: Separate commands (writes) from queries (reads) for complex domains.**
Why: CQRS improves performance, enables event sourcing, and simplifies complex business logic.

```rust
// Command side
#[derive(Debug)]
pub struct CreateUserCommand {
    pub email: String,
    pub name: String,
    pub role: Role,
}

#[async_trait]
pub trait CommandHandler<C> {
    type Result;
    type Error;
    async fn handle(&self, command: C) -> Result<Self::Result, Self::Error>;
}

// Query side
#[derive(Debug, Serialize)]
pub struct UserView {
    pub id: UserId,
    pub email: String,
    pub name: String,
    pub status: UserStatus,
}

#[async_trait]
pub trait QueryHandler<Q> {
    type Result;
    async fn handle(&self, query: Q) -> Result<Self::Result, Self::Error>;
}
```

## Security Implementation

**Rule: Hash passwords with Argon2 and implement secure JWT handling.**
Why: Prevents authentication bypass and protects against credential theft.

```rust
use argon2::{Argon2, PasswordHash, PasswordHasher, PasswordVerifier};
use jsonwebtoken::{encode, decode, Header, Algorithm, Validation};

pub struct AuthService {
    encoding_key: EncodingKey,
    decoding_key: DecodingKey,
    argon2: Argon2<'static>,
}

impl AuthService {
    pub fn hash_password(&self, password: &str) -> Result<String, AuthError> {
        let salt = SaltString::generate(&mut OsRng);
        let hash = self.argon2.hash_password(password.as_bytes(), &salt)
            .map_err(AuthError::HashingFailed)?;
        Ok(hash.to_string())
    }
    
    pub fn authenticate(&self, email: &str, password: &str, stored_hash: &str) -> Result<String, AuthError> {
        let parsed_hash = PasswordHash::new(stored_hash)
            .map_err(|_| AuthError::InvalidCredentials)?;
            
        self.argon2.verify_password(password.as_bytes(), &parsed_hash)
            .map_err(|_| AuthError::InvalidCredentials)?;
        
        let claims = Claims {
            sub: email.to_string(),
            exp: (Utc::now() + Duration::hours(24)).timestamp() as usize,
            roles: vec!["user".to_string()],
        };
        
        encode(&Header::default(), &claims, &self.encoding_key)
            .map_err(AuthError::TokenGeneration)
    }
}
```

**Rule: Sanitize and validate all inputs to prevent injection attacks.**
Why: Input validation prevents most security vulnerabilities including XSS and injection attacks.

```rust
use regex::Regex;
use once_cell::sync::Lazy;

static EMAIL_REGEX: Lazy<Regex> = Lazy::new(|| {
    Regex::new(r"^[^\s@]+@[^\s@]+\.[^\s@]+$").unwrap()
});

pub struct SecurityValidator;

impl SecurityValidator {
    pub fn sanitize_string(&self, input: &str, max_length: usize) -> Result<String, ValidationError> {
        if input.len() > max_length {
            return Err(ValidationError::InputTooLong { max: max_length });
        }
        
        // Remove dangerous characters
        let sanitized = input.chars()
            .filter(|c| c.is_alphanumeric() || " .,!?-_@".contains(*c))
            .collect();
        
        Ok(sanitized)
    }
    
    pub fn validate_email(&self, email: &str) -> Result<(), ValidationError> {
        if !EMAIL_REGEX.is_match(email) || email.len() > 254 {
            return Err(ValidationError::InvalidEmail);
        }
        Ok(())
    }
}
```

## Advanced Testing

**Rule: Use property-based testing to verify invariants across many inputs.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
