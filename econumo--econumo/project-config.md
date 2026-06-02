---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# Repository Guidelines

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

Econumo is a self-hosted personal finance and budgeting application. It consists of:
- **Backend**: Symfony 5.4 (PHP 8.2+) API with hexagonal architecture
- **Frontend**: Vue 3 + Quasar 2 SPA with TypeScript
- **Database**: SQLite (default) or PostgreSQL

## Development Commands

### Backend (PHP/Symfony)

All backend commands run inside Docker containers via `docker-compose`:

```bash
# Start application (includes migrations)
make up

# Stop application
make down

# Open shell in app container
make sh

# Run tests (recreates test DB and loads fixtures)
make test                    # All tests
make test ARGS='unit'       # Unit tests only
make test ARGS='functional' # Functional tests only

# Inside container shell (docker-compose exec -uwww-data app sh):
bin/console doctrine:migrations:migrate           # Run migrations
bin/console doctrine:fixtures:load                # Load fixtures
bin/console doctrine:database:create              # Create database
bin/console cache:clear                           # Clear cache
vendor/bin/codecept run unit --steps -v          # Run unit tests with verbose output
```

### Frontend (Vue/Quasar)

Frontend commands run on the host machine in the `web/` directory:

```bash
# Install dependencies (uses pnpm)
make install
# or: cd web && pnpm install

# Start development server (SPA mode)
make dev
# or: cd web && npm run dev

# Build for production
make bundle
# or: cd web && npm run build

# Run linter
make lint
# or: cd web && npm run lint

# Format code
cd web && npm run format
```

## Architecture

### Bundle Structure

The backend is organized into three Symfony bundles:

1. **EconumoBundle** (`src/EconumoBundle/`)
   - Core application logic
   - Contains all layers: Domain, Application, Infrastructure, UI
   - Handles accounts, transactions, budgets, categories, currencies, payees, tags

2. **EconumoToolsBundle** (`src/EconumoToolsBundle/`)
   - Utility commands and maintenance tools
   - Currency rate updates (OpenExchangeRates API)
   - Database migration tools (SQLite ↔ PostgreSQL)

### Layered Architecture (Hexagonal + DDD)

The codebase follows a strict layered architecture with dependency inversion:

```
Domain (Core Business Logic)
    ↓ depends on nothing
Application (Use Cases & Orchestration)
    ↓ depends on Domain
Infrastructure (Symfony/Doctrine Implementation)
    ↓ implements Domain interfaces
UI (HTTP Controllers & API)
    ↓ uses Application services
```

**Key Layer Responsibilities:**

- **Domain** (`src/EconumoBundle/Domain/`)
  - Entity models with business logic
  - Value objects (Id, DecimalNumber, AccountName, etc.)
  - Repository interfaces (no implementation)
  - Domain services (complex business rules)
  - Domain events

- **Application** (`src/EconumoBundle/Application/`)
  - Feature-organized services (AccountService, BudgetService, etc.)
  - Request/Result DTOs with V1 suffix
  - Assemblers (convert Domain ↔ DTOs)
  - Use case handlers

- **Infrastructure** (`src/EconumoBundle/Infrastructure/`)
  - `Doctrine/`: Repository implementations, ORM mappings, migrations, custom types
  - `Symfony/`: Forms, Messenger handlers, Mailer, Translation
  - `Auth/`: JWT authentication
  - `Datetime/`: DateTime services

- **UI** (`src/EconumoBundle/UI/`)
  - `Controller/Api/`: API controllers (one action per class)
  - `Middleware/`: Exception handling, language detection, API protection
  - `Service/`: Validators, response factories

### API Controller Pattern

Controllers follow a resource-based, single-action pattern:

```php
// Pattern: {Resource}/{SubResource?}/{Action}V1Controller
// Route: /api/v1/{resource}/{action}

class CreateAccountV1Controller extends AbstractController {
    #[Route(path: '/api/v1/account/create-account', methods: ['POST'])]
    public function __invoke(Request $request): Response {
        // 1. Validate via Form classes
        $this->validator->validate(CreateAccountV1Form::class, ...);
        // 2. Call Application service
        $result = $this->accountService->createAccount($dto, $userId);
        // 3. Return standardized response
        return ResponseFactory::createOkResponse($request, $result);
    }
}
```

**Validation**: Each controller has a corresponding Form class in `Validation/` subdirectory with declarative constraints.

### Frontend Architecture (Vue 3 + Quasar)

Directory structure in `web/src/`:

- **pages/**: Route pages (file-based routing)
- **components/**: Reusable Vue components
- **composables/**: Vue 3 Composition API hooks (e.g., `useApi`)
- **modules/api/v1/**: API client services (account.ts, budget.ts, etc.)
- **stores/**: Pinia state management (accounts, budgets, transactions, etc.)
- **router/**: Vue Router configuration
- **i18n/**: Internationalization (vue-i18n)

**State Management**: Pinia stores per feature with reactive state, getters, actions.

**API Integration**: `modules/api/v1/` contains typed API clients matching backend endpoints.

## Important Patterns

### Value Objects
- All IDs are wrapped in typed value objects (`AccountId`, `UserId`, etc.)
- Numbers use `DecimalNumber` for precision

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [econumo/econumo](https://github.com/econumo/econumo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
