---
trigger: always_on
description: This is a **three-service microservices architecture** for a book recommendation platform:
---

# Book Brain Microservices - Agent Guidelines

## Project Overview

This is a **three-service microservices architecture** for a book recommendation platform:

| Service | Port | Purpose | Tech Stack |
|---------|------|---------|-----------|
| **Gateway** | 4000 | API routing & request proxying | Express.js + http-proxy-middleware |
| **Book Brain** | 3000 | Core business logic (books, auth, reviews, etc.) | Express.js + PostgreSQL/MySQL |
| **Recommendation Service** | 5000 | ML-based book recommendations | Flask + Python |

**Architecture**: Client → Gateway (port 4000) → Book Service (internal:3000) or Recommend Service (internal:5000)

All services communicate through Docker network `app-network` using service names. A shared external PostgreSQL/MySQL database stores all data.

## Quick Start

### Local Development
```bash
# Book Brain Service
cd book_brain_service
npm install
npm run dev  # Port 3000, nodemon watches changes

# Gateway Service
cd gateway
npm install
npm run dev  # Port 8080

# Recommendation Service
cd recommend_service
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py  # Port 5000
```

### Docker Deployment
```bash
# Set environment variables in .env (see .env.example)
docker-compose up -d

# Clean rebuild
docker-compose down -v
docker-compose up -d --build
```

**See**: [book_brain_service/build.md](book_brain_service/build.md), [gateway/build.md](gateway/build.md), [recommend_service/build.md](recommend_service/build.md)

## Build & Push to Registry

Docker images published to `nguyenduc1603/` org on Docker Hub:

```bash
# Book Brain
docker build -t nguyenduc1603/book_brain:latest -t nguyenduc1603/book_brain:v1.0 ./book_brain_service
docker push nguyenduc1603/book_brain:latest

# Gateway
docker build -t nguyenduc1603/book-brain-gateway:latest ./gateway
docker push nguyenduc1603/book-brain-gateway:latest

# Recommendation Service
docker build -t nguyenduc1603/book-brain-ai-service:latest ./recommend_service
docker push nguyenduc1603/book-brain-ai-service:latest
```

## Code Conventions

### Book Brain Service (Node.js)

**Response Format** (standardized via [responseHelper.js](book_brain_service/src/utils/responseHelper.js)):
```javascript
{
  code: 200,
  status: 'success',
  message: 'Operation completed',
  data: { /* actual data */ },
  error: ''
}
```
**Note**: HTTP status is always 200; actual status is in the `code` field (intentional design).

**Architecture Pattern**:
- **Controllers** ([src/controllers/](book_brain_service/src/controllers/)) → **Services** ([src/services/](book_brain_service/src/services/)) → **Database** via parameterized queries
- Each feature has dedicated router, controller, and service (e.g., book.router.js → book.controller.js → book.service.js)

**Authentication**:
- JWT tokens via `Authorization: Bearer <token>` header
- Public routes (login, register) hardcoded in [authMiddleware.js](book_brain_service/src/middleware/authMiddleware.js)
- JWT_SECRET from environment variable

**Database Config** ([db.config.js](book_brain_service/src/configs/db.config.js)):
- Supports PostgreSQL (default, uses `$1, $2` placeholders) and MySQL (uses `?` placeholders)
- Select via `DB_TYPE` env var: `postgres` or `mysql`
- Connection pooling configured for both

**Validation**: [validation.js](book_brain_service/src/utils/validation.js) uses Joi schemas for input validation

**Logging**: Winston logger ([logger.js](book_brain_service/src/utils/logger.js)) with metadata support - use `logger.info()`, `logger.error()` with context

**Routes** (10 feature-based routers):
- `/api/v1/auth/*` - Authentication
- `/api/v1/books/*` - Book CRUD
- `/api/v1/reviews/*` - Book reviews
- `/api/v1/favorites/*` - Favorite books
- `/api/v1/rankings/*` - Book rankings
- `/api/v1/reading-history/*` - User reading history
- `/api/v1/book-notes/*` - Book annotations
- `/api/v1/categories/*` - Book categories
- `/api/v1/subscriptions/*` - Subscription management
- `/api/v1/notifications/*` - Push notifications (FCM)

### Gateway Service (Express.js)

**Purpose**: HTTP routing layer that proxies requests to downstream services

**Middleware Stack**:
- CORS (whitelist: `https://reset.nguyenduc.click`)
- Request logging via Morgan + Winston
- Error handler middleware
- Proxy middleware for service routing

**Health Check**: `GET /health` returns service status

**Route Mapping** ([src/routes/index.js](gateway/src/routes/index.js)):
- All incoming routes prefixed as `/api/v1/*`
- Forwards to `http://book_brain:3000` or `http://recommend:5000` (Docker internal URLs)
- Preserves Authorization and Cookie headers through proxy

### Recommendation Service (Flask + Python)

**Hybrid Recommendation Algorithm**:
- Combines content-based and collaborative filtering
- Configurable weights in [config.py](recommend_service/config.py)
- Caching: In-memory recommendations with TTL (default 120s)

**Endpoint**: `POST /recommend` - Takes user_id, returns ranked book list

**Database**: Connects to shared PostgreSQL via [db_service.py](recommend_service/services/db_service.py)

## Environment Variables

**Required** (create `.env` file):
```
# Database
DB_TYPE=postgres              # or mysql

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NguyenMinhDuc163/Book_brain_microservce](https://github.com/NguyenMinhDuc163/Book_brain_microservce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
