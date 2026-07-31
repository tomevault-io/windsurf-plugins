---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust-based social media backend API (mofu-backend) built with Axum web framework, SeaORM for database operations, and PostgreSQL as the primary database. The project follows a modular architecture with clear separation of concerns.

## Development Commands

### Main Application (Rust)
- `cargo run` - Start the development server
- `cargo build` - Build the application
- `cargo build --release` - Build optimized release version
- `cargo test` - Run tests
- `cargo check` - Quick syntax and type checking (preferred for development)
- `cargo clippy` - Run linter for code quality checks
- `cargo fmt` - Format code according to Rust standards

### Database Migrations (SeaORM)
- `cd migration && cargo run` - Run database migrations
- `cd migration && cargo run -- refresh` - Refresh all migrations

### Task Runner (Python/FastAPI + Celery)
Located in `tasks/` directory:
- `cd tasks && uv sync` - Install all dependencies including dev tools
- `cd tasks && uv pip install .` - Install only essential dependencies
- `cd tasks && uv run fastapi dev app/main.py` - Start task runner API (development)
- `cd tasks && uvicorn app.main:app --host 0.0.0.0 --port 7000` - Start task runner API (production)
- `cd tasks && python start_worker.py` - Start Celery worker for background tasks
- `cd tasks && python monitor_celery.py` - Start Celery Flower monitoring (requires flower package)
- `cd tasks && uv run ruff check .` - Run Python linter
- `cd tasks && uv run ruff check . --fix` - Fix auto-fixable linting issues
- `cd tasks && uv run ruff format .` - Format Python code

### Docker Development
- `docker-compose up` - Start all services (backend, tasks, redis, meilisearch, markdown service)
- `docker-compose up --build` - Rebuild and start all services
- `docker build -t mofumofu-backend .` - Build backend Docker image
- `docker run -p 8000:8000 --env-file docker.env mofumofu-backend` - Run backend container

## Architecture Overview

### Core Structure
- **API Layer**: `/src/api/v0/routes/` - RESTful endpoints organized by feature (auth, user, post, follow)
- **Service Layer**: `/src/service/` - Business logic implementation
- **Data Layer**: `/src/entity/` - Database entity definitions using SeaORM
- **DTOs**: `/src/dto/` - Data transfer objects for request/response handling
- **Middleware**: `/src/middleware/` - CORS, authentication, and request processing
- **Configuration**: `/src/config/` - Environment-based configuration management

### Key Components
- **Authentication**: JWT-based with access/refresh token pattern, OAuth support (Google, GitHub)
- **Database**: PostgreSQL with connection pooling via SeaORM
- **Search Engine**: Meilisearch integration for full-text search with automatic indexing
- **Background Tasks**: Celery with Redis broker for long-running operations (profile image upload, search indexing, etc.)
- **File Storage**: Cloudflare R2 (S3-compatible) for profile images and media with automatic WebP conversion
- **API Documentation**: Auto-generated Swagger UI at `/docs` endpoint (Rust) and `/tasks/docs` (Python)
- **Error Handling**: Centralized error management with proper HTTP status codes and structured error codes
- **Logging**: Structured logging with tracing crate and log rotation
- **Task Bridge**: Communication bridge between Rust backend and Python task runner via HTTP
- **Markdown Service**: Separate Bun/Elysia service for markdown rendering

### Database Entities
- `users` - User accounts and profiles
- `posts` - User-generated content
- `comments` - Post comments
- `follows` - User following relationships
- `hash_tags` - Content tagging system
- `user_refresh_tokens` - JWT refresh token management
- `likes` - Post like system
- `user_oauth_connections` - OAuth provider connections
- `system_events` - System activity logging

## Environment Setup

Copy `.env.example` to `.env` and configure:
- **Database connection** (PostgreSQL)
- **JWT secret** (generate with `openssl rand -base64 32`)
- **OAuth credentials** (Google, GitHub client IDs and secrets)
- **Cloudflare R2** (bucket name, access keys, public domain)
- **Redis connection** for caching and Celery broker
- **Meilisearch** host and API key
- **Server host/port** settings
- **CORS configuration**
- **Token expiration times**

## API Structure

All API endpoints are versioned under `/v0/`:
- `/v0/auth/*` - Authentication endpoints (OAuth, sign-in/out, token refresh, password management)
- `/v0/user/*` - User management (profiles, avatar/banner upload)
- `/v0/post/*` - Content management (CRUD, search, thumbnails, image upload)
- `/v0/follow/*` - Social following features (follow/unfollow, follower lists)
- `/v0/like/*` - Post like system (create/delete likes, like status)
- `/v0/hashtag/*` - Trending hashtags
- `/docs` - Swagger UI documentation

## Error Handling System

The application uses a structured error handling system with specific error codes:

### Error Categories
- **User errors**: `user:*` (not_found, unauthorized, invalid_password, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levish0/Mofumofu](https://github.com/levish0/Mofumofu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
