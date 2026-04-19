---
trigger: always_on
description: This is a Spring Boot 3.x application for personal finance management with the following key features:
---

# Finance Control - Cursor Rules

## Project Context
This is a Spring Boot 3.x application for personal finance management with the following key features:
- User authentication and authorization (JWT)
- Transaction management with categories
- Financial goals tracking
- Brazilian market integration (stocks, FIIs)
- Dashboard with analytics
- RESTful API with OpenAPI documentation

## Technology Stack
- Java 21
- Spring Boot 3.5.3
- Spring Security with JWT
- Spring Data JPA
- PostgreSQL
- Flyway for migrations
- Gradle for build management
- Docker for containerization
- OpenAPI/Swagger for API documentation

## Code Style & Standards

### Java & Spring Boot
- Use Java 21 features when applicable (records, sealed classes, pattern matching)
- Follow Spring Boot best practices and conventions
- Use constructor injection over field injection
- Implement proper exception handling with @ControllerAdvice
- Use @Valid for request validation
- Follow RESTful API design patterns

### Naming Conventions
- PascalCase for class names (e.g., UserController, TransactionService)
- camelCase for methods and variables (e.g., findUserById, isTransactionValid)
- ALL_CAPS for constants (e.g., MAX_RETRY_ATTEMPTS, DEFAULT_PAGE_SIZE)
- Use descriptive names that explain intent

### Package Structure
- Controllers in `*.controller` package
- Services in `*.service` package
- Repositories in `*.repository` package
- DTOs in `*.dto` package
- Models/Entities in `*.model` package
- Configuration in `shared.config` package

## Development Workflow

### Commits
- Always use Angular conventional commits pattern
- Use descriptive commit messages with scope
- Multiple commits divided by modules, scope, feature, etc.
- Example: `feat(transactions): add transaction reconciliation`

### Testing
- Write unit tests for all service methods
- Write integration tests for controllers
- Use @DataJpaTest for repository tests
- Maintain test coverage above 80%
- Use meaningful test names that describe the scenario

### Documentation
- Keep README.md updated with changes
- Update CHANGELOG.md for all significant changes
- Document API endpoints with OpenAPI annotations
- Add JavaDoc for complex business logic

## Security & Best Practices

### Security
- Never hardcode secrets, use environment variables
- Validate all user inputs
- Use proper authentication and authorization
- Implement CORS configuration
- Use HTTPS in production

### Error Handling
- Use descriptive error messages
- Implement proper HTTP status codes
- Log errors with appropriate levels
- Provide user-friendly error responses

### Performance
- Use pagination for large datasets
- Implement caching where appropriate
- Optimize database queries
- Use async processing for long-running tasks

## Database & Migrations

### Database
- Use Flyway for schema migrations
- Follow naming conventions for tables and columns
- Use proper indexes for performance
- Implement soft deletes where appropriate

### Migrations
- Create migrations for all schema changes
- Use descriptive migration names
- Test migrations on development data
- Never modify existing migrations

## API Design

### RESTful APIs
- Use proper HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Implement proper status codes
- Use consistent response formats
- Version APIs appropriately
- Document all endpoints with OpenAPI

### Request/Response
- Use DTOs for request/response objects
- Implement proper validation
- Use pagination for list endpoints
- Include proper error responses

## Docker & Deployment

### Docker
- Use multi-stage builds for optimization
- Use .dockerignore to exclude unnecessary files
- Use environment variables for configuration
- Keep images as small as possible

### Environment Configuration
- Use environment-specific properties
- Use @ConfigurationProperties for type-safe config
- Validate configuration on startup
- Use profiles for different environments

## Code Quality

### Tools
- Use Checkstyle for code style
- Use PMD for code analysis
- Use SpotBugs for bug detection
- Use SonarQube for quality gates

### Code Review
- Review all code changes
- Check for security vulnerabilities
- Verify test coverage
- Ensure documentation is updated

## Specific Project Rules

### Finance Control Specific
- Always validate financial calculations
- Use BigDecimal for monetary values
- Implement proper transaction boundaries
- Handle currency conversions properly
- Validate Brazilian market data

### Brazilian Market Integration
- Handle API rate limits
- Implement proper error handling for external APIs
- Cache market data appropriately
- Validate market data before storing

### Transaction Management
- Implement proper reconciliation logic
- Handle transaction categories correctly
- Validate transaction amounts
- Implement proper audit trails

## Terminal & Commands

### Preferred Tools
- Use `fd` instead of `find` command (install if not available)
- Use `ripgrep` instead of `grep` (install if not available)
- Use `bat` instead of `cat`
- Use `sudo` when elevated privileges are needed

### Development
- Don't ask for permission to proceed with steps
- If the approach is correct, continue without asking
- Use scripts for repetitive tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LucasSantana-Dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
