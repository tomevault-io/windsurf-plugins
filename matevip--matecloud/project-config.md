---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MateCloud is a **DDD microservice scaffold** built with Spring Boot 4.0.7 + Spring Cloud 2025.1.2 + Spring Cloud Alibaba + Dubbo 3.3.6. Java 21, MyBatis Plus, Sa-Token auth, Redisson, MapStruct.

Philosophy: **minimal common, each module does one thing well, starter = plug-and-play capability.**

## Development Commands

### Backend (Java / Maven)

```bash
# Full build (42 modules)
mvn clean install -DskipTests

# Build one module with dependencies
mvn clean install -pl mate-biz/mate-system -am -DskipTests

# Run a service (requires infra: Nacos + MySQL + Redis)
make infra-up                                       # start MySQL/Redis/RabbitMQ/Nacos/MinIO
cd mate-biz/mate-system && mvn spring-boot:run

# Tests
mvn test                                            # unit tests
mvn verify -Pintegration-test                       # integration (Testcontainers)

# Generate API docs (Smart-Doc, not Swagger)
mvn smart-doc:html -pl mate-biz/mate-system

# Database migration
java -jar mate-cli/target/mate-cli.jar db migrate --module mate-system
```

### Frontend (mate-ui)

```bash
cd mate-ui
pnpm install                  # pnpm monorepo (turbo orchestration)
pnpm dev                      # Vite dev server → http://localhost:3000

# Default dev credentials (created by AdminUserSeeder on first boot):
#   admin / admin123 / 13800138000
```

Frontend is a **pnpm workspace monorepo**: `apps/admin` (Vue 3 + Vite + Element Plus + TypeScript) with shared packages in `packages/{core,hooks,ui,utils}`. Vite proxies `/api` to `http://127.0.0.1:9010` (gateway).

### Docker / Compose

> **IMPORTANT — only `docker-compose` (v1, hyphenated) is supported in this environment. NEVER use `docker compose` (v2, space).** All compose commands must use the `docker-compose ...` binary, run from the repo root.

```bash
docker-compose build mate-system mate-ui   # rebuild changed images
docker-compose up -d mate-system mate-ui   # recreate (Flyway auto-runs new migrations on mate-system boot)
docker-compose logs -f mate-system         # watch logs
# Upgrade flow: git pull → docker-compose build <svc> → docker-compose up -d <svc>. Never `down -v` (wipes volumes).
```

### Makefile shortcuts

```bash
make build                    # mvn clean install -DskipTests
make build-module MODULE=mate-auth
make test                     # unit tests
make verify                   # integration tests
make infra-up / infra-down    # infrastructure only
make up / down                # full stack (docker-compose, v1 hyphenated binary)
```

### CLI tool (mate-cli)

```bash
java -jar mate-cli/target/mate-cli.jar new module mate-order --port 9060
java -jar mate-cli/target/mate-cli.jar service list
java -jar mate-cli/target/mate-cli.jar db describe mate_system --service mate-system
java -jar mate-cli/target/mate-cli.jar gen code --table mate_order --module mate-order --service mate-system
java -jar mate-cli/target/mate-cli.jar --mcp        # MCP server for Claude Code
```

## Architecture

### Module layout

```
matecloud/
├── mate-common/                  # Pure types (ZERO auto-config)
│   ├── mate-base/                # BaseEntity, Result, BizException, ErrorCode, SnowflakeUtil
│   └── mate-api/                 # Shared DTOs, Dubbo RPC interfaces, enums
├── mate-starters/                # 13 plug-and-play starters (7 core + 6 business)
├── mate-starters-contrib/        # 8 advanced starters (sharding, seata, sentinel, gray, flow, rule, ai, test)
├── mate-gateway/                 # API Gateway (WebFlux, port 9010)
├── mate-auth/                    # Auth service (port 9020)
├── mate-cli/                     # CLI + MCP server (Picocli)
├── mate-biz/
│   ├── mate-system/              # System management + DDD showcase (port 9030, RBAC + Dict + Config + AI)
│   └── mate-notice/              # Notification service (port 9050)
└── mate-ui/                      # Vue 3 frontend (Vite + Element Plus)
```

### Service ports

| Service | Port |
|---------|------|
| mate-gateway | 9010 |
| mate-auth | 9020 |
| mate-system | 9030 |
| mate-notice | 9050 |
| mate-ui (dev) | 3000 |

### DDD 4-layer package structure

Every business module under `vip.mate.{service}/` follows:

- **trigger/** — Inbound adapters: `controller/` (REST), `rpc/` (@DubboService), `event/` (@EventListener), `job/` (@XxlJob)
- **application/** — Orchestration: `command/` (writes, @Transactional), `query/` (reads, interface + impl/), `convertor/` (MapStruct)
- **domain/** — Core logic, **ZERO framework deps**: `model/{aggregate,entity,valobj}`, `service/`, `adapter/{repository,port}` (interfaces only), `event/`
- **infrastructure/** — Outbound adapters: `adapter/{repository,port}` (impls), `dao/` (MyBatis Plus mappers), `dao/po/` (persistent objects)
- **types/** — `exception/` (ErrorCode enum), `constant/`

### DDD rules

1. Domain layer has ZERO framework deps — no Spring/MyBatis annotations
2. Repository interface in domain, impl in infrastructure
3. Aggregate roots enforce invariants — state changes only through aggregate methods
4. CQRS — CommandService (writes) vs QueryService (reads), never mixed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matevip/matecloud](https://github.com/matevip/matecloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
