---
trigger: always_on
description: docker, infrastructure
---


# Infrastructure & DevOps Guidelines

## Docker & Containerization

### Dockerfile Best Practices

```dockerfile
# Multi-stage build pattern for production optimization
FROM node:18-alpine AS frontend-builder
WORKDIR /app
COPY frontend/package*.json ./
RUN npm ci --only=production
COPY frontend/ ./
RUN npm run build

FROM python:3.11-slim AS backend-base
WORKDIR /app
COPY backend/pyproject.toml ./
RUN pip install -e .

FROM backend-base AS backend-production
COPY backend/ ./
EXPOSE 8000
CMD ["gunicorn", "api:app", "--host", "0.0.0.0", "--port", "8000"]

# Security best practices
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

# Health check implementation
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

### Docker Compose Patterns

```yaml
# Production-ready docker-compose.yml
version: "3.8"

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_SUPABASE_URL=${SUPABASE_URL}
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - app-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    volumes:
      - ./backend/logs:/app/logs
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    restart: unless-stopped
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - backend
    restart: unless-stopped
    networks:
      - app-network

volumes:
  redis-data:

networks:
  app-network:
    driver: bridge
```

## Environment Management

### Environment Configuration

```bash
# .env.local (development)
NODE_ENV=development
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
DATABASE_URL=postgresql://user:pass@localhost:5432/suna_dev
REDIS_URL=redis://localhost:6379
LOG_LEVEL=debug

# .env.production
NODE_ENV=production
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
DATABASE_URL=postgresql://user:pass@prod-db:5432/suna_prod
REDIS_URL=redis://prod-redis:6379
LOG_LEVEL=info
SENTRY_DSN=your_sentry_dsn
```

### Tool Version Management (mise.toml)

```toml
[tools]
node = "18.17.0"
python = "3.11.5"
docker = "24.0.0"
docker-compose = "2.20.0"

[env]
UV_VENV = ".venv"
PYTHON_KEYRING_BACKEND = "keyring.backends.null.Keyring"
```

### Environment-Specific Scripts

```bash
#!/bin/bash
# scripts/start-dev.sh
set -e

echo "Starting development environment..."

# Check if required tools are installed
command -v docker >/dev/null 2>&1 || { echo "Docker is required but not installed. Aborting." >&2; exit 1; }
command -v docker-compose >/dev/null 2>&1 || { echo "Docker Compose is required but not installed. Aborting." >&2; exit 1; }

# Start services
docker-compose -f docker-compose.dev.yml up -d
echo "✅ Development services started"

# Wait for services to be healthy
echo "Waiting for services to be ready..."
sleep 10

# Run database migrations
docker-compose -f docker-compose.dev.yml exec backend python -m alembic upgrade head
echo "✅ Database migrations completed"

echo "🚀 Development environment is ready!"
echo "Frontend: http://localhost:3000"
echo "Backend: http://localhost:8000"
echo "Redis: localhost:6379"
```

## Deployment Strategies

### GitHub Actions CI/CD

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install backend dependencies
        run: |
          cd backend
          pip install -e .

      - name: Install frontend dependencies
        run: |
          cd frontend
          npm ci

      - name: Run backend tests
        run: |
          cd backend
          pytest

      - name: Run frontend tests
        run: |
          cd frontend
          npm run test

      - name: Lint code
        run: |
          cd backend && python -m black --check .
          cd frontend && npm run lint

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elhayatferdos67-ux/mytask.v2](https://github.com/elhayatferdos67-ux/mytask.v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
