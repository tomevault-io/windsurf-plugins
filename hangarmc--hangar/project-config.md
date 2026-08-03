---
trigger: always_on
description: Hangar is a **plugin repository platform** for Paper, Velocity, and Waterfall Minecraft server plugins. It is inspired by [Ore](https://github.com/SpongePowered/Ore), rebuilt from scratch with modern technologies, for PaperMC's needs.
---

# Hangar — Repository Guide for AI Agents

## Project Overview

Hangar is a **plugin repository platform** for Paper, Velocity, and Waterfall Minecraft server plugins. It is inspired by [Ore](https://github.com/SpongePowered/Ore), rebuilt from scratch with modern technologies, for PaperMC's needs.

- **Production**: https://hangar.papermc.io/
- **Staging**: https://hangar.papermc.dev/
- **License**: MIT

## Architecture

**Monorepo** with three main components:

| Component | Path | Technology | Purpose |
|-----------|------|-----------|---------|
| Backend | `backend/` | Java 21+ / Spring Boot 3.5 | REST API, business logic, database |
| Frontend | `frontend/` | TypeScript / Vue 3 / Nuxt 4 | SSR web application |
| Infrastructure | `docker/`, `chart/` | Docker Compose, Helm | Local dev and Kubernetes deployment |
| E2E Tests | `e2e/` | CodeceptJS / Playwright | Cross-browser end-to-end tests |

### Data Flow

```
Browser → Nuxt (SSR, port 3333) → Nitro proxy (/api/...) → Spring Boot (port 8080) → PostgreSQL
                                                                                    → MeiliSearch (search)
                                                                                    → RustFS / S3 (file storage)
```

---

## Backend (`backend/`)

### Stack

| Concern | Technology |
|---------|-----------|
| Framework | Spring Boot 3.5.7 |
| Language | Java 21+ (virtual threads enabled) |
| Build | Maven (`mvn`) |
| Database | PostgreSQL 15 via JDBI 3 (not JPA) |
| Migrations | Flyway (versioned + repeatable) |
| Search | MeiliSearch |
| Auth | Spring Security + JWT + TOTP + WebAuthn |
| Caching | Caffeine |
| Rate Limiting | Bucket4j |
| Clustering | JGroups (Kubernetes discovery) |
| Storage | Local filesystem or S3-compatible (RustFS in dev) |
| Error Tracking | Sentry |
| API Docs | SpringDoc OpenAPI |

### Package Structure (`io.papermc.hangar`)

```
├── HangarApplication.java          # Entry point
├── HangarComponent.java            # Base class for components (auth/config helpers)
├── components/                     # Feature modules (auth, jobs, webhooks, scheduler, etc.)
├── config/                         # Spring & Hangar configuration classes
├── controller/
│   ├── api/v1/                     # Public REST API (versioned)
│   └── internal/                   # Internal API used by frontend
├── db/
│   ├── dao/                        # JDBI Data Access Objects (SQL queries)
│   └── mappers/                    # JDBI row mappers (~68 mappers)
├── model/
│   ├── db/                         # Database entities
│   ├── api/                        # API request/response DTOs
│   ├── internal/                   # Internal models
│   └── loggable/                   # Audit logging models
├── service/
│   ├── api/                        # Services for public API
│   └── internal/                   # Core business logic (projects, versions, users, orgs, admin)
├── security/                       # Auth tokens, principals, permission annotations
├── exceptions/                     # Custom exception handling
└── util/                           # Utility classes
```

### Key Files

- **Config**: `backend/src/main/resources/application.yml`
- **Migrations**: `backend/src/main/resources/db/migration/` (V1.0.0 through V1.21.0+)
- **SQL Templates**: `backend/src/main/resources/io/papermc/hangar/db/dao/internal/`
- **Mail Templates**: `backend/src/main/resources/mail-templates/`

### Building & Testing

```bash
cd backend
mvn install                          # Build
mvn spring-boot:run                  # Run locally (needs Docker services)
mvn test                             # Run tests
```

Tests use **JUnit 5**, **Mockito**, and **TestContainers** (PostgreSQL). There are ~18 test classes covering services, controllers, components, and utilities.

### Important Patterns

- **JDBI, not JPA**: Database access uses JDBI with SQL templates and custom row mappers. There are no `@Entity` annotations or Hibernate.
- **Layered architecture**: Controller → Service → DAO
- **HangarComponent base class**: Many components extend `HangarComponent` which provides access to authentication, configuration, and action logging.
- **Custom security annotations**: Permission checks are done via annotations in `security/annotations/`.

---

## Frontend (`frontend/`)

### Stack

| Concern | Technology |
|---------|-----------|
| Framework | Nuxt 4.1 (Vue 3.5) |
| Language | TypeScript 5.9 |
| Package Manager | pnpm 10 |
| Runtime (prod) | Bun |
| Styling | UnoCSS (Tailwind-compatible utility classes) |
| State | Pinia (5 stores: auth, backendData, notification, prism, settings) |
| HTTP | Axios (proxied through Nitro server routes) |
| Forms | Vuelidate |
| i18n | vue-i18n / @nuxtjs/i18n (20+ languages, Crowdin managed) |
| Markdown | Marked + EasyMDE editor |
| Syntax Highlight | PrismJS |
| Charts | Chart.js + vue-chartjs |
| Error Tracking | Sentry |
| CAPTCHA | Cloudflare Turnstile |

### Directory Structure

```
frontend/
├── app/
│   ├── components/          # ~150+ Vue components (auto-imported)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HangarMC/Hangar](https://github.com/HangarMC/Hangar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
