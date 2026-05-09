---
trigger: always_on
description: <type>(<scope>): <description>
---

# Commit Message Conventions and Flow

## Commit Message Format

### Standard Format
```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Type Categories

#### Core Types
- **feat**: New feature for the user (e.g., new API endpoint, authentication method)
- **fix**: Bug fix for the user (e.g., security vulnerability, broken endpoint)
- **docs**: Documentation changes (README, API docs, comments)
- **style**: Code style changes (formatting, missing semicolons, no logic changes)
- **refactor**: Code refactoring (no new features or bug fixes)
- **test**: Adding or updating tests
- **chore**: Maintenance tasks (dependency updates, build scripts)

#### Security-Specific Types
- **security**: Security patches, vulnerability fixes, security improvements
- **deps**: Dependency updates (especially security-related)

### Scope Guidelines

#### Feature-Based Scopes
- **auth**: Authentication-related changes ([internal/auth/](mdc:internal/auth))
- **user**: User management functionality ([internal/user/](mdc:internal/user))
- **social**: Social login providers ([internal/social/](mdc:internal/social))
- **twofa**: Two-factor authentication ([internal/twofa/](mdc:internal/twofa))
- **email**: Email verification and notifications ([internal/email/](mdc:internal/email))
- **middleware**: HTTP middleware ([internal/middleware/](mdc:internal/middleware))
- **database**: Database operations and migrations ([internal/database/](mdc:internal/database))
- **redis**: Redis operations and caching ([internal/redis/](mdc:internal/redis))
- **log**: Activity logging system ([internal/log/](mdc:internal/log))

#### Technical Scopes
- **api**: API endpoints and handlers
- **models**: Database models ([pkg/models/](mdc:pkg/models))
- **dto**: Data transfer objects ([pkg/dto/](mdc:pkg/dto))
- **jwt**: JWT token handling ([pkg/jwt/](mdc:pkg/jwt))
- **config**: Configuration management ([internal/config/](mdc:internal/config))
- **docker**: Docker configuration ([Dockerfile](mdc:Dockerfile), [docker-compose.yml](mdc:docker-compose.yml))
- **build**: Build system ([Makefile](mdc:Makefile), Go modules)
- **ci**: CI/CD pipeline (.github workflows)

## Commit Message Examples

### Feature Development
```
feat(auth): add JWT token blacklisting for secure logout

- Implement Redis-based token blacklisting
- Add middleware check for blacklisted tokens
- Ensure immediate token invalidation on logout
- Include TTL for automatic cleanup

Closes #123
```

### Security Fixes
```
security(middleware): fix JWT token validation bypass

Critical security patch for authentication middleware that
allows unauthorized access when malformed tokens are provided.

- Validate token format before parsing
- Add proper error handling for invalid tokens
- Include rate limiting for failed attempts

BREAKING CHANGE: Invalid tokens now return 401 instead of 500
```

### API Changes
```
feat(user): add user profile update endpoint

- Add PUT /profile endpoint for user updates
- Implement email change verification flow
- Add validation for profile data
- Update Swagger documentation

Refs #456
```

### Database Changes
```
feat(models): add activity log model for audit tracking

- Create ActivityLog model with GORM tags
- Add database migration for activity_logs table
- Include indexing for performance optimization
- Add foreign key relationships

Migration: 20240121_create_activity_logs.sql
```

### Documentation Updates
```
docs(api): update Swagger annotations for 2FA endpoints

- Add comprehensive examples for TOTP setup
- Document recovery code generation flow
- Include error response schemas
- Update authentication requirements
```

### Testing
```
test(auth): add comprehensive JWT middleware tests

- Test token validation edge cases
- Add blacklist functionality tests
- Include performance benchmarks
- Mock Redis for isolated testing

Coverage: middleware/auth.go 95% -> 98%
```

### Dependency Updates
```
deps(security): update Go dependencies for security patches

- Update golang.org/x/crypto to v0.17.0
- Patch GORM to v1.25.5 for SQL injection fix
- Update Gin framework to v1.9.1

Addresses CVE-2023-45288, CVE-2023-45289
```

## Breaking Changes

### Format for Breaking Changes
```
feat(auth): redesign authentication flow for enhanced security

BREAKING CHANGE: Login endpoint now requires email verification.
All existing API clients must update to handle the new two-step
authentication process.

Migration guide:
1. Update login requests to include verification_required field
2. Implement email verification step before token issuance
3. Update error handling for unverified accounts

Closes #789
```

## Multi-Component Changes

### When Changes Affect Multiple Areas
```
feat(auth,user,middleware): implement role-based access control

- Add Role model and user-role relationships
- Update JWT claims to include user roles
- Add middleware for role-based endpoint protection
- Update user registration to assign default roles

Files modified:
- pkg/models/user.go
- pkg/models/role.go
- internal/middleware/auth.go
- internal/user/service.go

Closes #234, #235, #236
```

## Commit Flow Integration

### Pre-Commit Checklist
Before committing, ensure:
- [ ] Run `make fmt` for code formatting
- [ ] Run `make test` for test validation  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gjovanovicst/golang-auth-api](https://github.com/gjovanovicst/golang-auth-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
