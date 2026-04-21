---
trigger: always_on
description: This file provides context for AI assistants (Claude, Copilot, etc.) working on this repository. Keep it updated as the project evolves.
---

# CLAUDE.md — Jewel-Trade (B2B Jewel-Mall)

This file provides context for AI assistants (Claude, Copilot, etc.) working on this repository. Keep it updated as the project evolves.

---

## Project Overview

**Jewel-Trade** is a B2B precious-metals export company platform built for **Deco Indco**. It replaces manual Excel-based invoicing and messenger-based order management with a web-based Order Management System (OMS).

- **Type:** B2B e-commerce / OMS
- **Audience:** Overseas buyers and internal admins
- **Languages:** Korean (primary team language), English (single supported UI language)
- **Timeline:** 8 weeks; currently in early development (core features starting weeks 4–7)

### Business Context

| Problem (As-Is) | Solution (To-Be) |
|---|---|
| Orders received via messenger | Web-based order/quote request |
| Manual Excel invoice creation | Auto-generated PDF invoices |
| No order status visibility | Real-time order lifecycle tracking |

---

## Repository Structure

```
demo-repository/
├── demo/                         # Spring Boot backend (main application)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   ├── com/example/demo/
│   │   │   │   │   └── DemoApplication.java      # Entry point (@SpringBootApplication)
│   │   │   │   └── com/deco/demo/
│   │   │   │       └── SecurityConfig.java        # Spring Security configuration
│   │   │   └── resources/
│   │   │       └── application.properties         # Application settings
│   │   └── test/
│   │       └── java/com/example/demo/
│   │           └── DemoApplicationTests.java      # Context load test
│   ├── build.gradle                               # Gradle build configuration
│   ├── settings.gradle
│   └── gradlew / gradlew.bat                      # Gradle wrapper scripts
├── .github/
│   └── workflows/
│       ├── proof-html.yml                         # HTML validation CI
│       └── auto-assign.yml                        # Auto-assign issues/PRs to ksh322
├── index.html                                     # Static demo/landing page
├── package.json                                   # NPM config (Primer CSS dependency)
├── README.md                                      # Team guide (Korean)
├── PRD.md                                         # Product Requirements Document
├── erd.md                                         # Entity Relationship Diagram & site map
└── CLAUDE.md                                      # This file
```

**Package naming convention:**
- `com.example.demo` — Spring Boot application entry point (default generated)
- `com.deco.demo` — Company-specific configuration classes

---

## Tech Stack

### Backend
| Layer | Technology | Version |
|---|---|---|
| Language | Java | 17 |
| Framework | Spring Boot | 3.5.10 |
| Build tool | Gradle | 8.14.4 |
| Security | Spring Security | (Boot managed) |
| ORM | Spring Data JPA | (Boot managed) |
| Query | QueryDSL | 5.1.0 (jakarta) |
| Cache/Session | Spring Data Redis | (Boot managed) |
| API docs | SpringDoc OpenAPI (Swagger) | 2.8.1 |
| HTTP clients | Spring Cloud OpenFeign | 2025.0.1 |
| Validation | Spring Boot Validation | (Boot managed) |
| Utilities | Lombok | (Boot managed) |
| DB (prod) | PostgreSQL | (runtime) |
| DB (test) | H2 | (runtimeOnly) |

### Frontend (planned)
| Layer | Technology |
|---|---|
| Framework | TypeScript React |
| Rendering | Next.js (possible) |
| CSS | Primer CSS 17.0.1 (current static page) |

### Infrastructure
| Service | Purpose |
|---|---|
| AWS (Free-tier) / Oracle | Hosting |
| Docker | Containerization |
| EKS | Orchestration |
| Railway | Alternative deployment |
| Redis | JWT blacklist + session management |
| PostgreSQL | Primary database |

---

## Build & Run Commands

All commands run from the `demo/` directory.

```bash
# Build the project
./gradlew build

# Run all tests
./gradlew test

# Run the application (dev mode with hot reload)
./gradlew bootRun

# Clean build artifacts and QueryDSL generated sources
./gradlew clean

# Build without running tests
./gradlew build -x test
```

QueryDSL Q-classes are auto-generated to `demo/src/main/generated/` during compilation and cleaned by `./gradlew clean`.

---

## Configuration

### application.properties
Currently minimal. Environment-specific values should be injected at runtime via environment variables or Spring profiles.

Required environment variables (not yet in `.env.example` — create one):
```
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/jeweltrade
SPRING_DATASOURCE_USERNAME=
SPRING_DATASOURCE_PASSWORD=
SPRING_DATA_REDIS_HOST=
SPRING_DATA_REDIS_PORT=6379
SPRING_MAIL_HOST=
SPRING_MAIL_USERNAME=
SPRING_MAIL_PASSWORD=
JWT_SECRET=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

### Spring Security
File: `demo/src/main/java/com/deco/demo/SecurityConfig.java`

- CSRF is **disabled** (convenience for testing — re-enable before production)
- Swagger endpoints (`/v3/api-docs/**`, `/swagger-ui/**`) are public
- All other endpoints require authentication
- Successful form login redirects to `/swagger-ui/index.html`

---

## MVP Feature Scope

### In Scope
| Feature | Description |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Deco-Project-26-01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
