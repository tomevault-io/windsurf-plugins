---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "Centro Tía Glenda" - a healthcare management system built with Flask (Python) that manages medical center operations including patients, staff, specialties, therapy sessions, and user authentication. The system uses PostgreSQL as the database and provides a REST API with Swagger documentation.

## Development Commands

### Running the Application
```bash
python app.py
```
The Flask app runs on port 5000 by default, configurable via PORT environment variable.

### Testing
```bash
# Run all tests using the advanced test runner
python tests/test_api_complete_master.py

# Run specific module tests
python tests/test_autenticacion_api.py
python tests/test_pacientes_api.py
python tests/test_personal_api.py
python tests/test_especialidades_api.py
python tests/test_usuarios_api.py
python tests/test_tutores_api.py
python tests/test_personas_api.py
python tests/test_roles_api.py
python tests/test_sesiones_terapia_api.py
python tests/test_sesiones_pedagogicas_api.py
python tests/test_documentos_pacientes_api.py

# Unit tests
python tests/test_units.py
```

### Environment Setup
```bash
# Install dependencies
pip install -r requirements.txt

# Activate virtual environment (if using venv)
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate
```

## Architecture

### Directory Structure
- `src/api/`: Core API implementation
  - `Components/`: Data access layer (database interactions)
  - `Service/`: Business logic layer
  - `routes/`: Route definitions and HTTP handling
- `src/utils/`: Utility modules
  - `database/`: Database connection and operations
  - `general/`: Configuration, logging, authentication, security
- `tests/`: Comprehensive test suite with advanced test runner
- `static/swagger.json`: API documentation
- `docs/`: Documentation directory (currently empty)

### API Architecture Pattern
The project follows a 3-layer architecture:
1. **Routes** (`api_routes.py`): HTTP request handling and routing
2. **Services** (`*Service.py`): Business logic and validation
3. **Components** (`*Component.py`): Data access and database operations

### Key Modules
- **Authentication**: JWT-based with role-based access control (@token_required, @admin_required)
- **Database**: PostgreSQL with custom connection handling (DataBaseHandle class)
- **Logging**: Comprehensive logging system with daily log files in src/utils/general/LOGS/
- **Security**: Password hashing, JWT tokens, input validation
- **Testing**: Advanced test runner with progress bars, colored output, and detailed statistics
- **Dashboard**: Role-based dashboard system with admin, therapist, and pedagogue views
- **Multi-Center Configuration**: Support for multiple healthcare centers with isolation

## Configuration

### Database Configuration
Database settings are managed through:
1. Environment variables (highest priority)
2. `src/utils/general/config.cfg` file
3. Default values in `src/utils/general/config.py`

Key environment variables:
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- `JWT_SECRET`
- `AMBIENTE` (DEVELOPMENT/PRODUCTION)

### API Endpoints Structure
- `/api/test`: API health check
- `/api/test-db`: Database connectivity test
- `/api/login`, `/api/logout`, `/api/verify-token`, `/api/me`: Authentication
- `/api/personas`: Person management
- `/api/usuarios`: User management (admin-only)
- `/api/personal`: Staff management
- `/api/especialidades`: Medical specialties
- `/api/pacientes`: Patient management
- `/api/tutores`: Guardian/tutor management
- `/api/roles`: Role management
- `/api/sesiones-terapia`: Therapy session management with cronograma generation
- `/api/sesiones-pedagogicas`: Pedagogical session management with class scheduling
- `/api/dashboard/*`: Dashboard endpoints with role-based data (admin, therapist, pedagogue views)
- `/api/centros-disponibles`: Available centers configuration
- `/docs/`: Swagger UI documentation
- `/health`: Application health endpoint

## Development Practices

### Testing Philosophy
The project uses a sophisticated testing framework (`tests/utils/advanced_test_runner.py`) that provides:
- Color-coded progress bars
- Detailed timing statistics
- ETA calculations
- Retry mechanisms for failed tests
- Export capabilities for test results

### Security Implementation
- JWT tokens for stateless authentication
- Role-based authorization (regular users vs admin)
- Password hashing with bcrypt
- Input validation and sanitization
- CORS configuration for frontend integration

### Logging Strategy
- Separate error and info logs by date
- Location: `src/utils/general/LOGS/`
- Automatic rotation by day
- Integration throughout the application

### Database Patterns
- Custom DataBaseHandle class for connection management with specific methods:
  - `getRecords()`: For SELECT queries (returns dict/list)
  - `getRecordsWithStatus()`: For SELECT with detailed error handling and status info
  - `ExecuteNonQuery()`: For INSERT/UPDATE/DELETE without return values
  - `ExecuteInsert()`: For INSERT with ID return (limited use)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mickaell22/ProjectTiaGlendaBackend](https://github.com/Mickaell22/ProjectTiaGlendaBackend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
