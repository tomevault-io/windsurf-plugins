---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**ShareShelf**

ShareShelf is a community-powered tool library web application that lets neighbors borrow and lend rarely used tools and equipment. Instead of buying items used only a few times a year, members share resources within their community — saving money and reducing waste. The app handles user registration, item listings, borrowing requests, reviews/ratings, and trust scoring.

**Core Value:** Users can discover and borrow tools from neighbors in their community, with a trusted borrowing workflow that protects both lenders and borrowers.

### Constraints

- **Tech Stack**: Spring Boot 3.4.x / Kotlin / Next.js 15 / PostgreSQL — locked
- **Deployment**: Railway (backend) + Vercel (frontend) — infrastructure decided
- **Auth Model**: JWT-based stateless auth — not changing
- **Language**: Backend code in Kotlin, frontend in TypeScript
- **Testing**: JUnit 5 + MockK (backend), Vitest + RTL (frontend), Playwright (E2E)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- **Kotlin 2.1.0** — Backend application logic, JPA entities, controllers, services. Located in `backend/src/main/kotlin/com/shareshelf/`.
- **TypeScript ~5.7** — Frontend application logic, component props, API type definitions. Located in `frontend/src/`.
- **Java 21** — Underlying JVM runtime. No hand-written Java source files found; all backend code is Kotlin compiled to JVM bytecode via `jvmToolchain(21)`.
## Runtime
- **JVM** (eclipse-temurin:21-jre-jammy in production) — Backend runs as a Spring Boot fat JAR (`shareshelf-0.0.1-SNAPSHOT.jar`).
- **Node.js** (via Vercel) — Frontend is built statically or server-side rendered by Next.js.
- **Gradle 8.12** — Kotlin DSL (`settings.gradle.kts`, `build.gradle.kts`). Wrapper configured in `backend/gradle/wrapper/gradle-wrapper.properties`.
- **npm** — `frontend/package.json` present, no lockfile committed (neither `package-lock.json` nor `yarn.lock` or `pnpm-lock.yaml` detected). Dependency versions are semver-ranged.
## Frameworks
- **Spring Boot 3.4.3** — Java/Kotlin web framework. Application entry point at `backend/src/main/kotlin/com/shareshelf/ShareShelfApplication.kt`.
- **Next.js 15 (App Router)** — React meta-framework. Configuration at `frontend/next.config.ts`. All pages are client components under `frontend/src/app/`.
- **React 19** — UI component library. Client components only (no Server Components used in pages found).
- **JUnit 5** (via `spring-boot-starter-test`) — Test runner.
- **MockK 1.13.14** — Kotlin-native mocking library.
- **SpringMockK 4.0.2** — Spring Boot integration for MockK.
- **Spring Security Test** — `@WithMockUser` / `SecurityMockServerRequest` support.
- **Spring Boot DevTools** — Hot reload during development.
- **Kotlin JPA plugin** — `kotlin("plugin.jpa")` for no-arg constructors on entities.
- **Kotlin AllOpen plugin** — `kotlin("plugin.allopen")` to open JPA-annotated classes.
- **Kotlin Spring plugin** — `kotlin("plugin.spring")` to open `@Configuration`/`@Bean` classes.
## Key Dependencies
- **jjwt 0.12.6** (`io.jsonwebtoken:jjwt-api`) — JWT token generation and validation at `backend/src/main/kotlin/com/shareshelf/auth/JwtTokenProvider.kt`. Well-integrated, uses `Keys.hmacShaKeyFor()` with HMAC-SHA.
- **Springdoc OpenAPI 2.8.5** (`springdoc-openapi-starter-webmvc-ui:2.8.5`) — Swagger UI at `/swagger-ui.html` and OpenAPI spec at `/api-docs`. Enabled in dev, disabled in railway profile (`application-railway.yml`).
- **Axios 1.7.9** — HTTP client for frontend API calls. Singleton instance at `frontend/src/lib/api.ts` with request/response interceptors for JWT injection and 401 handling.
- **Flyway** (`flyway-core`, `flyway-database-postgresql`) — Database migrations. Migrations at `backend/src/main/resources/db/migration/V1`-`V5`. `ddl-auto: validate` ensures schema matches migrations.
- **PostgreSQL JDBC Driver** (`org.postgresql:postgresql`) — Database connectivity. Runtime scope.
## Configuration
- Backend uses Spring profiles: `default` (dev at `application.yml` + `application-dev.yml` active by default?), and `railway` (`application-railway.yml` activated in Docker by `SPRING_PROFILES_ACTIVE=railway`).
- Frontend uses `NEXT_PUBLIC_API_URL` environment variable (defaults to `/api` proxy in `frontend/src/lib/api.ts`).
| Variable | Purpose | Set in |
|---|---|---|
| `JWT_SECRET` | HMAC-SHA key for signing tokens | `application.yml` (fallback dev value) |
| `DB_PASSWORD` | Local dev database password | `application.yml` (fallback `shareshelf_dev`) |
| `PGHOST`, `PGPORT`, `PGDATABASE`, `PGUSER`, `PGPASSWORD` | Railway Postgres connection | `application-railway.yml` |
| `PORT` | Server port on Railway | `application-railway.yml` (defaults 8080) |
| `NEXT_PUBLIC_API_URL` | Backend URL for frontend API calls | Frontend env at deploy |
- Backend: `backend/build.gradle.kts` (Gradle Kotlin DSL).
- Frontend: `frontend/next.config.ts`, `frontend/postcss.config.mjs`.
- Docker: `Dockerfile` at project root (multi-stage build using `gradle:8.12-jdk21` then `eclipse-temurin:21-jre-jammy`).
## Platform Requirements
- JDK 21 (tested with Eclipse Temurin).
- Kotlin 2.1.0 compiler (via Gradle plugin).
- Node.js 18+ (for frontend dev server).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [winnaingsoe6666/ShareShelf](https://github.com/winnaingsoe6666/ShareShelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
