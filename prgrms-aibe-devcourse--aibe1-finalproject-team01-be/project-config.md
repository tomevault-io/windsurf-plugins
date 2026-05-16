---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
And You should respond in Korean.

## Common Development Commands

### Building and Testing
```bash
# Build the project (includes JOOQ code generation)
./gradlew build

# Run tests with coverage
./gradlew test

# Run a single test class
./gradlew test --tests "ClassName"

# Run a specific test method
./gradlew test --tests "ClassName.methodName"

# Generate test coverage report
./gradlew jacocoTestReport

# Verify test coverage meets requirements (minimum 50%)
./gradlew jacocoTestCoverageVerification

# Clean build artifacts (including generated JOOQ classes)
./gradlew clean

# Generate JOOQ classes only (requires database to be running)
./gradlew generateJooqClasses
```

### Database Setup
```bash
# Start MySQL database (required for JOOQ code generation and local development)
docker-compose -f docker/mysql/docker-compose.yml up -d

# Start Redis cache
docker-compose -f docker/redis/docker-compose.yml up -d

# View container status
docker-compose -f docker/mysql/docker-compose.yml ps

# Stop containers
docker-compose -f docker/mysql/docker-compose.yml down
```

### Running the Application
```bash
# Run with development profile
./gradlew bootRun --args='--spring.profiles.active=dev'

# Run with local profile (default)
./gradlew bootRun --args='--spring.profiles.active=local'
```

## High-Level Architecture

This is a **Spring Boot 3.5.0** backend application for the Amateurs community platform, using:

### Core Stack
- **Java 17** with Spring Boot 3.5.0
- **JOOQ** for type-safe SQL queries (primary data access layer)
- **JPA/Hibernate** for entity management
- **MySQL 8.0** as primary database
- **Redis** for caching and session management
- **MongoDB** for chat message storage

### AI Integration
- **LangChain4j** with **Google Gemini** for AI-powered features
- **Qdrant** vector database for post recommendations
- Custom AI profile analysis and content recommendation system

### Communication Layer
- **WebSocket + STOMP** for real-time chat
- **Server-Sent Events (SSE)** for real-time notifications
- **Spring Security** with JWT authentication

### Key Architectural Patterns
1. **Domain-Driven Design**: Organized by business domains (auth, post, ai, etc.)
2. **Event-Driven Architecture**: Uses Spring Events for decoupled components
3. **Repository Pattern**: Separate JPA and JOOQ repositories
4. **Strategy Pattern**: For alarm creation and report handling
5. **AOP**: Custom annotations for alarm triggers and post metadata validation

## Project Structure

### Core Packages
- `annotation/`: Custom annotations with AOP aspects
  - `alarmtrigger/`: Auto-generates alarms using AOP
  - `checkpostmetadata/`: Validates post metadata
- `config/`: Configuration classes for security, database, AI, etc.
- `controller/`: REST API endpoints organized by domain
- `domain/dto/`: Data transfer objects for API communication
- `repository/`: Data access layer (both JPA and JOOQ implementations)
- `service/`: Business logic layer
- `utils/`: Utility classes

### Database Architecture
- **Primary DB (MySQL)**: User data, posts, comments, authentication
  - Schema versioning with **Flyway** migrations in `src/main/resources/db/migration/`
  - Initial schema: `V1__init_schema.sql`
- **Cache (Redis)**: Session storage, popular posts cache, view counts
- **Document DB (MongoDB)**: Chat messages and rooms
- **Vector DB (Qdrant)**: Post embeddings for AI recommendations

### AI System
- **Post Recommendation**: Uses vector similarity with Qdrant
- **Content Analysis**: Gemini-powered text analysis
- **User Profiling**: AI-generated user profiles based on activity
- **Smart Verification**: OCR + AI for automatic student verification

## Important Development Notes

### JOOQ Usage
- JOOQ is the **primary query mechanism** for complex queries and reporting
- Generated classes are in `src/generated/` (excluded from git)
- **IMPORTANT**: Database must be running before generating JOOQ classes
  1. Start MySQL: `docker-compose -f docker/mysql/docker-compose.yml up -d`
  2. Generate classes: `./gradlew generateJooqClasses`
  3. If build fails with JOOQ errors, regenerate classes after ensuring DB is running
- Use `DSLContext` for building type-safe queries
- Follow patterns in existing `*JooqRepository` classes
- Refer to `docs/jooq-guide.md` for detailed usage examples

### Testing Strategy
- **Integration Tests**: Use `@SpringBootTest` with Testcontainers (MySQL)
- **Unit Tests**: Mock external dependencies
- **Coverage Requirement**: Minimum 50% instruction and branch coverage
- **Test Database**: Testcontainers with MySQL 8.0 for integration tests
- **Redis**: Embedded Redis for testing
- **MongoDB**: Embedded MongoDB for testing
- Excluded from coverage: config, domain, exception, handler, repository, utils packages

### Security & Authentication
- **JWT Tokens**: Access + Refresh token pattern
- **OAuth2**: Kakao social login integration
- **Role-based Access**: ADMIN, USER roles with fine-grained permissions
- **CORS**: Configured for frontend integration

### Monitoring & Observability
- **OpenTelemetry**: Distributed tracing configured

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prgrms-aibe-devcourse/AIBE1-FinalProject-Team01-BE](https://github.com/prgrms-aibe-devcourse/AIBE1-FinalProject-Team01-BE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
