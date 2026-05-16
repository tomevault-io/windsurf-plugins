---
trigger: always_on
description: APPLY deployment best practices when working with Docker and infrastructure files
---

globs: Dockerfile, docker-compose.yml, .github/workflows/*, config/**/*
alwaysApply: false
---
# Deployment and Operations

**Rule Priority:** Core Architecture  
**Activation:** Always Active  
**Scope:** Production deployment, monitoring, scaling, and operational procedures

## Deployment Architecture Overview

SYMindX implements a **containerized, cloud-native deployment strategy** designed for scalability, reliability, and ease of maintenance across different environments.

### Deployment Stack Structure
```
┌─────────────────────────────────────────────────────────────┐
│                  SYMindX Deployment Stack                    │
├─────────────────────────────────────────────────────────────┤
│  Load Balancer    │  Container Orchestration │  Monitoring  │
│  ├─ NGINX/Caddy   │  ├─ Docker Compose       │  ├─ Grafana │
│  ├─ SSL/TLS       │  ├─ Kubernetes (opt)     │  ├─ Prometheus │
│  └─ Rate Limiting │  ├─ Health Checks        │  ├─ Loki    │
│                   │  └─ Auto-scaling         │  └─ Alerts  │
├─────────────────────────────────────────────────────────────┤
│  Application Layer                                          │
│  ├─ Mind Agents Service ├─ Website Service ├─ Docs Service │
│  ├─ Extension Services  ├─ API Gateway     ├─ MCP Server   │
│  └─ Worker Services     └─ WebSocket       └─ Background   │
├─────────────────────────────────────────────────────────────┤
│  Data Layer                                                 │
│  ├─ PostgreSQL/Supabase ├─ Redis Cache ├─ File Storage     │
│  ├─ Vector Database     ├─ Log Storage ├─ Backup Storage   │
│  └─ Configuration Store └─ Metrics DB  └─ Artifact Store   │
└─────────────────────────────────────────────────────────────┘
```

## Docker Configuration

### Multi-Stage Dockerfile
```dockerfile
# mind-agents/Dockerfile
FROM oven/bun:1-alpine AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile --production

# Build stage
FROM base AS build
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun run build
RUN bun run test

# Production stage
FROM base AS runtime
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 mindagent

COPY --from=deps --chown=mindagent:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=mindagent:nodejs /app/dist ./dist
COPY --from=build --chown=mindagent:nodejs /app/package.json ./

USER mindagent

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD bun run healthcheck

ENV NODE_ENV=production
CMD ["bun", "run", "start"]
```

### Website Dockerfile
```dockerfile
# website/Dockerfile
FROM node:20-alpine AS base
WORKDIR /app

# Dependencies
FROM base AS deps
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Build stage
FROM base AS build
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage with NGINX
FROM nginx:alpine AS runtime
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose Configuration
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Core mind agents service
  mind-agents:
    build: 
      context: ./mind-agents
      dockerfile: Dockerfile
    container_name: symindx-mind-agents
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    ports:
      - "8080:8080"
    volumes:
      - ./config:/app/config:ro
      - mind-agents-data:/app/data
      - mind-agents-logs:/app/logs
    networks:
      - symindx-network
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "bun", "run", "healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  # Website service
  website:
    build:
      context: ./website
      dockerfile: Dockerfile
    container_name: symindx-website
    restart: unless-stopped
    ports:
      - "3000:80"
    environment:
      - NEXT_PUBLIC_API_URL=http://mind-agents:8080
    networks:
      - symindx-network
    depends_on:
      - mind-agents

  # Documentation site
  docs-site:
    build:
      context: ./docs-site
      dockerfile: Dockerfile
    container_name: symindx-docs
    restart: unless-stopped
    ports:
      - "3001:80"
    networks:
      - symindx-network

  # PostgreSQL database
  postgres:
    image: pgvector/pgvector:pg16
    container_name: symindx-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-symindx}
      POSTGRES_USER: ${POSTGRES_USER:-symindx}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./config/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    ports:
      - "5432:5432"
    networks:
      - symindx-network
    healthcheck:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
