---
trigger: always_on
description: > This file is the AI equivalent of README.md. If you are an AI agent working on this repository, read this file first.
---

# AGENTS.md — Context for AI Agents

> This file is the AI equivalent of README.md. If you are an AI agent working on this repository, read this file first.

---

## What is sE2EEnd?

**sE2EEnd** is an open-source, end-to-end encrypted (E2EE) file transfer solution. The core invariant: **the server never sees encryption keys or plaintext data**. Encryption happens entirely in the browser using the native Web Crypto API (AES-256-GCM). The server stores only ciphertext and metadata.

Key capabilities:
- E2EE file transfers (AES-256-GCM, client-side only)
- Enterprise authentication via Keycloak (OAuth2 / OIDC)
- Password-protected transfers
- Per-transfer download limits and expiration dates
- Instant revocation
- Admin dashboard
- Multi-language UI (EN, FR)

**License:** AGPL-3.0

---

## Architecture Overview

```
Browser (React)
  ├── Encrypts files locally (Web Crypto API)
  ├── Authenticates via Keycloak (JWT Bearer)
  └── Sends ciphertext to backend

Backend (Spring Boot)
  ├── Stores ciphertext + file metadata
  ├── Validates JWTs (via Keycloak JWKS)
  └── Exposes REST API at /api/v1

Keycloak
  └── Identity Provider (OAuth2/OIDC, realm: se2eend)

PostgreSQL
  └── Persists sends + file metadata
```

**Key architectural rule:** The backend must never decrypt data. Any change that causes the server to handle plaintext or encryption keys violates the zero-knowledge guarantee.

---

## Tech Stack

| Layer       | Technology                                                                |
|-------------|---------------------------------------------------------------------------|
| Backend     | Java 21, Spring Boot 3.5.6, Maven 3.9                                     |
| Database    | PostgreSQL 18.1, Spring Data JPA / Hibernate                              |
| Migrations  | Flyway (SQL migrations in `db/migration/`)                                |
| Security    | Spring Security, OAuth2 Resource Server, JWT                              |
| API Docs    | SpringDoc OpenAPI (disabled by default, `SWAGGER_ENABLED=true` to enable) |
| Frontend    | React 19, TypeScript 5, Vite 7                                            |
| Styling     | Tailwind CSS 3.4                                                          |
| Auth (FE)   | @react-keycloak/web + keycloak-js                                         |
| HTTP Client | Axios (with JWT interceptor)                                              |
| i18n        | i18next (EN + FR)                                                         |
| Crypto      | Web Crypto API (browser-native, no lib)                                   |
| IdP         | Keycloak 26.5                                                             |
| Containers  | Docker + Docker Compose                                                   |
| CI/CD       | GitHub Actions → GHCR                                                     |

---

## Repository Structure

```
sE2EEnd/
├── backend/                          # Spring Boot application
│   └── src/main/
│       ├── java/fr/se2eend/backend/
│       │   ├── controller/           # REST endpoints
│       │   ├── service/              # Business logic
│       │   ├── model/                # JPA entities
│       │   ├── repository/           # Spring Data repos
│       │   ├── dto/                  # Request/response DTOs
│       │   ├── config/               # Security, CORS, OpenAPI
│       │   ├── storage/              # File storage abstraction
│       │   ├── scheduler/            # Cleanup cron (daily at 2 AM)
│       │   └── exception/            # Custom exceptions
│       └── resources/
│           └── db/migration/         # Flyway SQL migrations (V1__, V2__, ...)
├── frontend/core/                    # React application
│   └── src/
│       ├── features/                 # upload/, download/, dashboard/, admin/, profile/
│       ├── components/               # Shared components
│       ├── lib/
│       │   ├── crypto.ts             # ALL encryption logic lives here
│       │   └── sendKeysDB.ts         # Local key persistence (IndexedDB)
│       ├── services/api.ts           # Axios client + JWT interceptor
│       ├── contexts/                 # ThemeContext
│       ├── hooks/
│       └── locales/en/ + locales/fr/ # i18n translations
├── keycloak/
│   ├── realm-config/                 # Realm import JSON
│   └── themes/                       # Custom Keycloak theme CSS
├── docs/
│   └── encryption-flow.mermaid       # E2EE flow diagram (read this)
├── .github/workflows/
│   ├── backend.yml                   # CI: Maven build + tests
│   ├── frontend.yml                  # CI: lint + build
│   └── release.yml                   # Release: Docker → GHCR
├── docker-compose.yml                # Dev environment
├── .env.example                      # All required env vars
├── README.md                         # Human-facing documentation
├── CONTRIBUTING.md
├── SECURITY.md
└── CODE_OF_CONDUCT.md
```

---

## Running the Project Locally

### Prerequisites
- Java 21
- Node.js 25+
- Docker & Docker Compose

### 1. Configure environment

```bash
cp .env.example .env
# Edit .env with your values
```

### 2. Start infrastructure (PostgreSQL + Keycloak)

```bash
docker-compose up -d

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sE2EEnd/sE2EEnd](https://github.com/sE2EEnd/sE2EEnd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
