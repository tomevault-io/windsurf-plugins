---
trigger: always_on
description: > **This file contains context about the MovieMind API project for the AI assistant.**
---

# MovieMind API - Project Context

> **This file contains context about the MovieMind API project for the AI assistant.**
> 
> It is automatically loaded by Cursor IDE when the "Include CLAUDE.md in context" option is enabled in settings.

---

## 🎯 Project Overview

MovieMind API is a RESTful API for generating and storing unique descriptions of movies, series, and actors using AI technology. The project creates original, AI-generated content instead of copying content from IMDb or TMDb.

---

## 🏗️ Technology Stack

### Backend
- **Framework:** Laravel 12
- **PHP:** 8.2+
- **Database:** PostgreSQL (production and tests)
- **Cache:** Redis
- **Queue:** Laravel Horizon (asynchronous processing)
- **AI Integration:** OpenAI API (gpt-4o-mini)

### Frontend
- **Stack:** Vue 3, Vite, Tailwind CSS (SPA in `frontend/`)
- **CI:** `.github/workflows/frontend.yml` (lint + build on `frontend/**`)

### Development Tools
- **Tests:** PHPUnit (Feature Tests + Unit Tests)
- **Formatting:** Laravel Pint (PSR-12)
- **Static analysis:** PHPStan (level 5)
- **Security:** GitLeaks (secret detection)
- **Documentation:** OpenAPI/Swagger
- **Local Environment:** Docker Compose (mandatory - see `.cursor/rules/090-docker-development.mdc`)

---

## 🐳 Local Development

### Docker is Mandatory

**ALWAYS use Docker for local development. NEVER use `php artisan serve` directly on host.**

**Quick Start:**
```bash
# Start all services
docker compose up -d

# Setup application (first time)
docker compose exec php composer install
docker compose exec php php artisan key:generate
docker compose exec php php artisan migrate --seed

# Access application
# API: http://localhost:8000
```

**Run commands inside PHP container:**
```bash
docker compose exec php php artisan migrate
docker compose exec php php artisan test
docker compose exec php vendor/bin/pint
```

**Run tests (mandatory: inside Docker):** Testy używają PostgreSQL (`DB_HOST=db`). Na hoście „db” nie istnieje, więc **nie uruchamiaj** `cd api && composer test` na hoście – dostaniesz błędy połączenia z bazą. Uruchamiaj testy **w kontenerze**:
```bash
# Z roota repozytorium (zalecane):
docker compose exec php composer test

# Lub bezpośrednio:
docker compose exec php bash run-tests.sh
```
Testy uruchamiane są **równolegle** (`--parallel`) – Laravel tworzy osobne bazy testowe (np. `moviemind_test_1`, `moviemind_test_2`) dla każdego procesu. Użytkownik bazy (np. `moviemind` w Dockerze) musi mieć prawo tworzenia baz (CREATEDB).
Z katalogu `api/` możesz też wejść do kontenera i tam uruchomić `composer test`. Pierwsze uruchomienie może trwać dłużej (migracje). Jeśli `docker compose exec php php artisan test` długo nic nie wyświetla – poczekaj ok. 30–60 s (RefreshDatabase + migracje), albo uruchom z `--verbose`.

**Why Docker is required:**
- Matches production environment (PostgreSQL, Redis, PHP-FPM, Nginx)
- Prevents "works on my machine" issues
- Ensures consistency between local, CI, and production
- See `.cursor/rules/090-docker-development.mdc` for full details

---

## 📁 Project Structure

### Main Structure
```
api/                          # Laravel application (backend)
├── app/
│   ├── Enums/               # Enumerations (Language, EntityType, etc.)
│   ├── Events/              # Laravel Events
│   ├── Features/            # Feature-based code
│   ├── Helpers/             # Helper functions
│   ├── Http/
│   │   ├── Controllers/     # API Controllers
│   │   ├── Requests/        # Request validators
│   │   └── Resources/        # API Resources
│   ├── Jobs/                # Queue Jobs (ShouldQueue)
│   ├── Listeners/           # Event Listeners
│   ├── Models/              # Eloquent Models
│   ├── Repositories/        # Repository pattern
│   └── Services/            # Business logic services
├── config/                  # Laravel configuration
├── database/
│   ├── migrations/          # Database migrations
│   └── seeders/             # Seeders
├── routes/
│   └── api.php              # Route definitions
└── tests/
    ├── Feature/             # Feature tests (API endpoints)
    └── Unit/                # Unit tests (classes, services)

frontend/                    # Vue 3 + Vite + Tailwind (SPA for API)
├── src/
│   ├── App.vue
│   ├── main.js
│   ├── style.css            # Tailwind directives
│   └── config.js            # VITE_API_BASE_URL etc.
├── public/
├── index.html
├── package.json
├── vite.config.js           # Dev proxy /api → backend
├── tailwind.config.js
└── README.md                # Setup, dev, build
```

The frontend is a separate app that consumes the MovieMind API. Run it with `cd frontend && npm run dev` (see `frontend/README.md`). CI: `.github/workflows/frontend.yml` runs lint and build on changes under `frontend/`.

---

## 🗄️ Data Model

### Main Tables

**Movies**
- `id` (PK)
- `title`
- `release_year`
- `director`
- `genres` (array)
- `default_description_id` (FK)

**Movie Descriptions**
- `id` (PK)
- `movie_id` (FK)
- `locale` (pl-PL, en-US, etc.)
- `text` (AI-generated content)
- `context_tag` (modern, critical, humorous)
- `origin` (GENERATED/TRANSLATED)
- `ai_model` (gpt-4o-mini)
- `created_at`

**Actors & Bios**
- Similar structure to Movies/Descriptions
- `actors` - basic actor data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
