---
trigger: always_on
description: Omni-Stack is a microservices scaffolding platform built with Spring Boot 4 + Vue 3, organized as a monorepo with a Maven multi-module backend and a standalone npm frontend.
---

# AGENTS.md

## Project Identity

Omni-Stack is a microservices scaffolding platform built with Spring Boot 4 + Vue 3, organized as a monorepo with a Maven multi-module backend and a standalone npm frontend.

| Layer       | Technology                                  | Version        |
|-------------|---------------------------------------------|----------------|
| JDK         | OpenJDK                                     | 25             |
| Backend     | Spring Boot                                 | 4.0.6          |
| Cloud       | Spring Cloud                                | 2025.1.1       |
| Cloud Alibaba | Spring Cloud Alibaba                     | 2025.1.0.0     |
| Gateway     | Spring Cloud Gateway Server (WebFlux)       | 5.0.1          |
| Service Discovery / Config | Nacos Server              | v3.1.1         |
| Circuit Breaker / Flow Control | Sentinel Dashboard            | 1.8.8          |
| Frontend    | Vue 3 + TypeScript                          | 3.5.35 / 5.9.3 |
| Bundler     | Vite 8 (Rolldown)                           | 8.0.14         |
| UI Framework| Element Plus                                | 2.14.0         |
| State       | Pinia                                       | 3.0.4          |
| Router      | Vue Router                                  | 5.0.7          |
| Node.js     | Node.js LTS                                 | >= 22.12.0     |

## System Truth

Architecture, patterns, API contracts, and core flows are documented in `docs/`. **Read those first.** This file contains only execution rules and build commands.

| Document | Purpose |
|----------|---------|
| `docs/architecture.md` | System boundaries, module map, data flow, RBAC permission system, constraints |
| `docs/api-contract.md` | Response format, error codes, pagination, naming |
| `docs/backend-patterns.md` | Java layering, validation, exceptions, logging, security & data permission, OOP rules |
| `docs/frontend-patterns.md` | Vue/TS patterns, state management, routing, permission control, component conventions |
| `docs/core-flows.md` | End-to-end traces of login (password + captcha, GitHub social, Gitee social, device code), RBAC functional permission (Flow 5), data permission (Flow 6), XSS defense (Flow 8) |

## Entry Points

**Backend:**
- Auth service: `omni-backend/omni-auth/src/main/java/com/omni/auth/AuthApplication.java`
- Base service: `omni-backend/omni-base/src/main/java/com/omni/base/BaseApplication.java`
- Gateway: `omni-backend/omni-gateway/src/main/java/com/omni/gateway/GatewayApplication.java`
- Common library: `omni-backend/omni-common/src/main/java/com/omni/common/`
- Common core (POJO): `omni-backend/omni-common-core/src/main/java/com/omni/common/core/`
- Common MyBatis-Plus starter: `omni-backend/omni-common-mybatis/src/main/java/com/omni/common/mybatis/`
- Common Redis starter (blocking): `omni-backend/omni-common-redis/src/main/java/com/omni/common/redis/`
- Common Redis starter (reactive): `omni-backend/omni-common-redis-reactive/src/main/java/com/omni/common/redis/reactive/`

**Frontend:**
- App bootstrap: `omni-frontend/src/main.ts`
- Router: `omni-frontend/src/router/index.ts`
- Shared types: `omni-frontend/src/types/api.ts`

**Configuration:**
- Auth config: `omni-backend/omni-auth/src/main/resources/application.yml`
- Gateway config: `omni-backend/omni-gateway/src/main/resources/application.yml`
- Base config: `omni-backend/omni-base/src/main/resources/application.yml`
- Vite config: `omni-frontend/vite.config.ts`

**RBAC & Permission:**
- Data scope filter: `omni-backend/omni-auth/src/main/java/com/omni/auth/security/DataScopeResolveFilter.java`
- Data permission handler: `omni-backend/omni-auth/src/main/java/com/omni/auth/security/DataPermissionHandlerImpl.java`
- Data scope context: `omni-backend/omni-auth/src/main/java/com/omni/auth/security/DataScopeContext.java`
- MyBatis-Plus config: `omni-backend/omni-auth/src/main/java/com/omni/auth/config/MyBatisPlusConfig.java`
- Dynamic menu controller: `omni-backend/omni-auth/src/main/java/com/omni/auth/controller/MenuController.java`
- Permission store: `omni-frontend/src/stores/permission.ts`
- v-permission directive: `omni-frontend/src/directives/permission.ts`

**XSS Defense:**
- XSS config provider SPI: `omni-backend/omni-common-core/src/main/java/com/omni/common/core/security/XssConfigProvider.java`
- XSS sanitizer: `omni-backend/omni-common/src/main/java/com/omni/common/security/xss/XssSanitizer.java`
- XSS filter + auto-config: `omni-backend/omni-common/src/main/java/com/omni/common/security/xss/XssFilter.java`
- XSS config implementation: `omni-backend/omni-auth/src/main/java/com/omni/auth/security/XssConfigProviderImpl.java`
- XSS config controller: `omni-backend/omni-auth/src/main/java/com/omni/auth/controller/XssConfigController.java`
- XSS management page: `omni-frontend/src/views/system/xssconfig/index.vue`
- Gateway security headers: `omni-backend/omni-gateway/src/main/java/com/omni/gateway/config/SecurityHeadersFilter.java`

## Build & Run Commands

### Prerequisites

- **JAVA_HOME** must be set to JDK 25 before running any Maven commands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wang-baohai/Omni-Stack](https://github.com/wang-baohai/Omni-Stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
