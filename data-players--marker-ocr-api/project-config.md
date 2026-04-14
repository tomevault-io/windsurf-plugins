---
trigger: always_on
description: - **One Layer, One Responsibility**: Each layer has a single, well-defined responsibility
---

# Marker-OCR-API Project Rules

## Architecture Principles

### Core Principles
- **One Layer, One Responsibility**: Each layer has a single, well-defined responsibility
- **Separation of Concerns**: Business logic, API layer, and data handling are strictly separated
- **Dependency Injection**: Services are injected rather than imported directly
- **Clean Architecture**: Dependencies point inward toward the business logic

### Project Structure
```
backend/
├── app/
│   ├── core/           # Configuration, exceptions, logging
│   │   ├── config.py   # Pydantic Settings with environment variables
│   │   ├── exceptions.py  # Custom exception classes
│   │   └── logger.py   # Structured logging setup
│   ├── api/            # FastAPI routes and HTTP layer
│   │   ├── dependencies.py  # Dependency injection (singletons with LRU cache)
│   │   └── routes/     # Route modules (health, documents)
│   ├── services/       # Business logic layer
│   │   ├── document_parser.py  # Real Marker service
│   │   ├── document_parser_mock.py  # Mock for tests
│   │   ├── file_handler.py  # Real file operations
│   │   ├── file_handler_mock.py  # Mock for tests
│   │   ├── redis_service.py  # Redis cache/task queue
│   │   └── marker_log_handler.py  # Marker-specific log handling
│   ├── models/         # Pydantic models for requests/responses
│   │   ├── request_models.py  # Request validation models
│   │   ├── response_models.py  # Response models
│   │   └── enums.py    # Enumeration types
│   └── utils/          # Pure utility functions
│       └── validators.py  # Input validation utilities
├── Dockerfile          # Production/dev image (with all ML dependencies)
├── Dockerfile.test-modelFree  # Lightweight test image (no ML models)
├── Dockerfile.test-FullStack  # Full test image (with ML models)
├── requirements-base.txt  # Core dependencies (NIVEAU 2)
├── requirements-minimal.txt  # Minimal test dependencies (NIVEAU 1)
└── requirements-models.txt  # ML model dependencies (NIVEAU 3)

frontend/
├── src/
│   ├── components/     # Reusable UI components
│   ├── services/       # API communication layer
│   ├── utils/          # Pure utility functions
│   └── pages/          # Page components
├── Dockerfile          # Production image
├── Dockerfile.dev      # Development image with hot reloading
└── tests/              # Frontend tests

tests/
├── backend/
│   ├── modelFree/      # Tests without ML dependencies (< 1s)
│   │   ├── conftest.py  # Fixtures with mock services
│   │   └── test_services/  # Service unit tests
│   └── FullStack/      # Integration tests with ML models
│       └── test_api_url_upload_integration.py
├── frontend/           # Component tests
└── local/              # Manual/development test scripts

docker-compose.dev.yml  # Development with hot reloading
docker-compose.test-modelFree.yml  # Fast tests (modelFree)
docker-compose.test-FullStack.yml  # Full integration tests
```

### Repository Separation
- **This repository (Marker-OCR-API)**: Development and testing only
  - Contains: dev/test docker-compose files, source code, test infrastructure
  - Does NOT contain: Production configuration, Traefik, SSL/TLS setup
- **Production repository (Marker-OCR-API-prod)**: Production deployment
  - Contains: Traefik reverse proxy, SSL/TLS, production docker-compose
  - Managed separately with different deployment workflow

## Code Standards

### Language Rules
- **Comments and Code**: Always in English
- **Chat Communication**: In French
- **Variable/Function Names**: English, descriptive, following conventions

### Python Backend
- Use FastAPI for API layer with async/await
- Pydantic Settings for configuration management (environment variables)
- Dependency injection pattern with singleton services (LRU cache)
- Lifespan management: Model initialization at startup, cleanup at shutdown
- Hot reload support: Handle CancelledError gracefully during development
- Type hints mandatory
- Max function length: 20 lines
- Max class length: 100 lines
- Follow PEP 8
- Structured logging with context (request IDs, operation tracking)
- Custom exception hierarchy (BaseAPIException) with proper HTTP status codes

### Frontend
- Modern JavaScript/TypeScript
- Component-based architecture
- Separation of concerns between UI and business logic
- Error boundaries for robust error handling

### Testing Architecture

#### Two-Tier Testing Strategy
1. **modelFree Tests** (Fast, < 1s):
   - Use `Dockerfile.test-modelFree` with `requirements-minimal.txt`
   - Mock services (`*_mock.py`) replace heavy dependencies
   - No ML model loading required
   - Located in `tests/backend/modelFree/`
   - Use `docker-compose.test-modelFree.yml`

2. **FullStack Tests** (Integration with ML):
   - Use `Dockerfile.test-FullStack` with all requirements
   - Real Marker library and ML models
   - For integration testing with actual PDF processing
   - Located in `tests/backend/FullStack/`
   - Use `docker-compose.test-FullStack.yml`

#### Testing Standards
- **Mandatory**: Unit tests for all business logic in services/
- **Mandatory**: Integration tests for API endpoints
- **Mandatory**: Component tests for frontend
- Test coverage minimum: 80%
- Use pytest for Python, Jest for JavaScript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/data-players) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
