---
trigger: always_on
description: - FastAPI: Web framework for building the API
---

# Evo AI - Project Rules and Structure

## Main Technologies
- FastAPI: Web framework for building the API
- SQLAlchemy: ORM for database interaction
- Alembic: Database migration system
- PostgreSQL: Main database
- Pydantic: Data validation and serialization
- Uvicorn: ASGI server for application execution
- Redis: Cache and session management
- JWT: Secure token authentication
- Bcrypt: Secure password hashing
- SendGrid: Email service for notifications
- Jinja2: Template engine for email rendering

## Project Structure
```
src/
├── api/
│   ├── __init__.py        # Package initialization
│   ├── admin_routes.py    # Admin routes for management interface
│   ├── agent_routes.py    # Routes to manage agents
│   ├── auth_routes.py     # Authentication routes (login, registration)
│   ├── chat_routes.py     # Routes for chat interactions with agents
│   ├── client_routes.py   # Routes to manage clients
│   ├── mcp_server_routes.py # Routes to manage MCP servers
│   ├── session_routes.py  # Routes to manage chat sessions
│   └── tool_routes.py     # Routes to manage tools for agents
├── config/
│   ├── database.py        # Database configuration
│   └── settings.py        # General settings
├── core/
│   ├── middleware.py      # API Key middleware (legacy)
│   └── jwt_middleware.py  # JWT authentication middleware
├── models/
│   └── models.py          # SQLAlchemy models
├── schemas/
│   ├── schemas.py         # Main Pydantic schemas
│   ├── chat.py            # Chat schemas
│   ├── user.py            # User and authentication schemas
│   └── audit.py           # Audit logs schemas
├── services/
│   ├── agent_service.py   # Business logic for agents
│   ├── agent_runner.py    # Agent execution logic
│   ├── auth_service.py    # JWT authentication logic
│   ├── audit_service.py   # Audit logs logic
│   ├── client_service.py  # Business logic for clients
│   ├── email_service.py   # Email sending service
│   ├── mcp_server_service.py # Business logic for MCP servers
│   ├── session_service.py # Business logic for chat sessions
│   ├── tool_service.py    # Business logic for tools
│   └── user_service.py    # User management logic
├── templates/
│   ├── emails/
│   │   ├── base_email.html           # Base template with common structure and styles
│   │   ├── verification_email.html   # Email verification template
│   │   ├── password_reset.html       # Password reset template  
│   │   ├── welcome_email.html        # Welcome email after verification
│   │   └── account_locked.html       # Security alert for locked accounts
├── tests/
│   ├── __init__.py        # Package initialization
│   ├── api/
│   │   ├── __init__.py        # Package initialization
│   │   ├── test_auth_routes.py # Test for authentication routes
│   │   └── test_root.py       # Test for root endpoint
│   ├── models/
│   │   ├── __init__.py        # Package initialization
│   │   ├── test_models.py     # Test for models
│   ├── services/
│   │   ├── __init__.py        # Package initialization
│   │   ├── test_auth_service.py # Test for authentication service
│   │   └── test_user_service.py # Test for user service
└── utils/
    ├── logger.py          # Logger configuration
    └── security.py        # Security utilities (JWT, hash)
```

## Code Standards

### Language Requirements
- **ALL code comments, docstrings, and logging messages MUST be written in English**
- Variable names, function names, and class names must be in English
- API error messages must be in English
- Documentation (including inline comments) must be in English
- Code examples in documentation must be in English
- Commit messages must be in English

### Project Configuration
- Dependencies managed in `pyproject.toml` using modern Python packaging standards
- Development dependencies specified as optional dependencies in `pyproject.toml`
- Single source of truth for project metadata in `pyproject.toml`
- Build system configured to use setuptools
- Pytest configuration in `pyproject.toml` under `[tool.pytest.ini_options]`
- Code formatting with Black configured in `pyproject.toml`
- Linting with Flake8 configured in `.flake8`

### Schemas (Pydantic)
- Use `BaseModel` as base for all schemas
- Define fields with explicit types
- Use `Optional` for optional fields
- Use `Field` for validations and default values
- Implement `Config` with `from_attributes = True` for models
- Use `EmailStr` for email validation

### Services
- Error handling with `SQLAlchemyError`
- Consistent logging with messages in English
- Strong typing with `Optional` for null returns
- Documentation with docstrings
- Rollback in case of error
- Standardized returns
- Use transactions for multiple operations

### Email Templates
- All email templates extend a base template
- Templates written in English
- Use Jinja2 templating system
- Consistent styling using a common base template
- Responsive design for mobile compatibility
- Clear call-to-action buttons
- Fallback mechanisms for failed template rendering

### Routes
- Appropriate status codes (201 for creation, 204 for deletion)
- Error handling with `HTTPException`
- Error messages in English
- Pagination for list endpoints
- Input validation with schemas
- JWT authentication for all protected routes
- Use of asynchronous functions with `async def`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvolutionAPI/evo-ai](https://github.com/EvolutionAPI/evo-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
