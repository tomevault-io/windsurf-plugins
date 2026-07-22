---
trigger: always_on
description: Universal AI agent instructions for Gravitee Access Management (AM).
---

# AGENTS.md — Gravitee Access Management

Universal AI agent instructions for Gravitee Access Management (AM).
Consumed by: OpenAI Codex, GitHub Copilot, Cursor, Gemini/Jules, Windsurf, Zed, Aider, and others.

---

# Agent Guide

If you are an AI agent operating in this repository:
- You MUST read this file fully before making changes.
- You MUST follow the rules defined here.
- If any instruction conflicts with other files, this file takes precedence.

## 1. Quick Reference Commands

### Prerequisites

- Java 25+, Maven 3.6+, Node.js >= 20.11.1, Yarn 4.1.1, Docker & Docker Compose

### Backend (Java / Maven)

```bash
# Full build (with tests)
mvn clean install

# Build without tests
mvn install -DskipTests

# Build excluding UI module
mvn install -pl '!gravitee-am-ui'

# Clean (excluding UI)
mvn clean -pl '!gravitee-am-ui'
```

### Frontend (Angular / TypeScript)

All commands run from `gravitee-am-ui/`.

```bash
# Install dependencies
yarn install

# Dev server
yarn serve                # or from root: make startUi

# Lint (all: ESLint, Stylelint, license, Prettier)
yarn lint
yarn lint:fix             # auto-fix

# Individual linters
yarn lint:eslint
yarn lint:styles
yarn lint:license
yarn prettier

# Build
yarn build                # development
yarn prod                 # production
```

### API / Contracts

```bash
# OpenAPI spec location
docs/mapi/openapi.yaml

# Validate OpenAPI spec
cd gravitee-am-test && npm install
npx @openapitools/openapi-generator-cli validate -i ../docs/mapi/openapi.yaml

# Regenerate Management API SDK (TypeScript)
cd gravitee-am-test && npm run update:sdk:mapi -- <MANAGEMENT_API_URL>
```

### Build Local Stack

One command builds (or pulls) AM and its dependencies. See `docker/local-stack/README.md`
and the `local-stack` skill for the full reference.

```bash
cd docker/local-stack

# Build from the current code state and start (lean: gateway, management, mongo, smtp, wiremock)
./local-stack.sh up

# Everything the jest/playwright suites need + Console UI on :4200
./local-stack.sh up --full

# Start a specific released/nightly version instead of building (no Maven)
./local-stack.sh up --version 4.12.x-latest --ui

# Tear down
./local-stack.sh down
```

### Running Tests

See **Section 8** for full test documentation (all test types, commands, and CI pipeline details).

```bash
# Pre-PR verification
mvn clean install
cd gravitee-am-ui && yarn lint && yarn test

# Quick backend iteration
mvn clean install -pl gravitee-am-service -DskipTests
mvn test -pl gravitee-am-service

# Quick frontend iteration
cd gravitee-am-ui && yarn lint:fix && yarn test
```

---

## 2. Project Context

### What Is Gravitee AM?

Gravitee Access Management is an identity and access management (IAM) platform supporting OAuth 2.0, OpenID Connect, SAML, and MFA. It provides a Management API, a Gateway, and a web UI for managing security domains, applications, identity providers, and policies.

### Tech Stack

| Layer | Technology                                                                                                |
|-------|-----------------------------------------------------------------------------------------------------------|
| Backend | Java 25, RxJava 3, Vert.x, Spring                                                                         |
| Frontend | Angular, TypeScript                                                                                       |
| Build | Maven (backend), Yarn 4.1.1 (frontend)                                                                    |
| Databases | MongoDB, PostgreSQL, MySQL, MariaDB, SQL Server                                                           |
| Migrations | Liquibase (JDBC), programmatic (MongoDB)                                                                  |
| API spec | OpenAPI 3 (`docs/mapi/openapi.yaml`)                                                                      |
| Tests | JUnit 5, Mockito, Testcontainers (backend); Jest (integration); Newman (contract); helm-unittest (charts) |
| Containers | Docker, Docker Compose                                                                                    |

### Key Paths

| Path | Description |
|------|-------------|
| `gravitee-am-management-api/` | Management API module |
| `gravitee-am-gateway/` | Gateway module |
| `gravitee-am-service/` | Core business logic |
| `gravitee-am-model/` | Domain model |
| `gravitee-am-repository/` | Repository implementations |
| `gravitee-am-dataplane/` | Reactive data access layer (MongoDB/JDBC) |
| `gravitee-am-identityprovider/` | Identity provider plugins |
| `gravitee-am-factor/` | MFA factor plugins |
| `gravitee-am-ui/` | Angular frontend |
| `gravitee-am-test/` | Integration tests (Jest) |
| `gravitee-am-test/GUIDELINES.md` | Integration test guidelines |
| `postman/collections/` | Postman/Newman API contract tests |
| `helm/` | Helm charts and tests (`helm/tests/`) |
| `docs/mapi/openapi.yaml` | OpenAPI 3 spec for Management API |
| `docs/agent-standards/` | Agent rules, skills, and templates |
| `Makefile` | Docker/local development commands |

### Conventions

- Use **US English** for comments, logs, and user-facing text.
- **Search for similar implementations first** before writing new code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gravitee-io/gravitee-access-management](https://github.com/gravitee-io/gravitee-access-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
