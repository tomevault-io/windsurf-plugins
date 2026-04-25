---
trigger: always_on
description: Copy to project-rule.mdc; pick ONE tier. Java / Spring Boot (REST APIs, business backends).
---

# Java / Spring Boot project-rule (template)

Replace `{{APP_NAME}}`, base packages (`com.company.app`), and infrastructure names. **Keep only one tier** in the active rule.

---

## How to choose a tier

| Tier | When to use |
|------|-------------|
| **Small** | Internal API, BFF, CRUD service, small team |
| **Medium** | Customer-facing SaaS backend, standard SLAs |
| **Big** | Enterprise, multi-team, compliance, high scale |

---

## Tier: Small project

### S1. Scope

- **Single deployable** JAR; **layered packages** `controller` → `service` → `repository` are enough.
- **Spring Boot** with **Spring Web**; **Spring Data JPA** if relational DB; **H2** or **Testcontainers** for dev/test.
- **Validation** with **`jakarta.validation`** on DTOs.

### S2. Layout

```
src/main/java/com/company/{{APP_NAME}}/
├── {{AppName}}Application.java
├── config/
├── controller/
├── service/
├── repository/
├── domain/              # entities (JPA) optional here or under model/
└── dto/
```

### S3. API style

- **REST** with consistent **HTTP verbs** and **status codes**.
- **Problem Details** (`spring-boot-starter-web` + `ErrorController` or `@ControllerAdvice`) for errors — even a simple JSON `{ "message" }` is fine if consistent.

### S4. Config and secrets

- **`application.yml`** profiles: `dev`, `prod`; **secrets** from **env** or **vault** — never committed.

### S5. Testing

- **`@WebMvcTest`** for controllers; **`@DataJpaTest`** for repos if JPA; smoke **`@SpringBootTest`** optional.

### S6. Checklist (small)

- [ ] Health endpoint (`/actuator/health` if actuator enabled).
- [ ] Input validation on all external inputs.
- [ ] SQL injection safe (JPA parameterized).

---

## Tier: Medium project

### M1. Architecture

- **Clear boundaries**: controllers thin; **services** own transactions and orchestration; **repositories** data access only.
- **DTOs** for API; **entities** for persistence; **mapping** explicit (MapStruct or manual) — avoid exposing entities in JSON.
- **Package by feature** acceptable when it improves cohesion:

```
feature/
├── order/
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   └── dto/
```

### M2. Spring practices

- **`@Transactional`** on service layer with correct **readOnly** for queries.
- **`@ControllerAdvice`** for **domain exceptions** → HTTP mapping.
- **OpenAPI** (`springdoc-openapi`) generated and kept in sync with controllers.

### M3. Persistence

- **Flyway** or **Liquibase** for migrations; **no** manual DDL in prod.
- **Indexes** and **N+1** awareness; **`@EntityGraph`** / **fetch joins** where needed.

### M4. Security

- **Spring Security** with **JWT** or **OAuth2 Resource Server**; **role/scope** checks on sensitive endpoints.
- **CORS** explicit; **rate limiting** (Bucket4j, gateway) for public APIs.

### M5. Observability

- **Micrometer** + **Prometheus** or vendor APM; **correlation ID** filter (MDC).
- **Structured logging** (JSON) in prod; **log levels** per package.

### M6. Testing

- **Testcontainers** for integration tests against real DB/Redis.
- **Contract tests** optional (Spring Cloud Contract) if many consumers.

### M7. Java style

- **Java 17+** LTS; **records** for DTOs where appropriate.
- **Immutability** for DTOs; **Builder** only when complexity warrants.
- **Checkstyle** or **Spotless** in CI.

### M8. Profiles, error tracking, and feature flags (medium baseline)

Production-oriented medium services treat these as **standard**.

- **Profiles (`dev` / `staging` / `prod`)**: **Spring profiles** (`spring.profiles.active`) with **`application-{profile}.yml`**; **no** production secrets in repo — **env**, **vault**, or **secret manager**. **Datasource URLs**, feature integrations, and **log levels** differ per profile; document how CI runs each profile for tests.
- **Crash and performance**: JVM **metrics** via **Actuator** + **Micrometer** (Prometheus stack) or vendor **APM** (Datadog, New Relic, Elastic). **Error tracking** (**Sentry** for Java, or log-based alerts) with **release** / **git SHA** tags. Define **SLOs** for error rate and **p95** latency on **staging** before prod deploys.
- **Feature flags**: **LaunchDarkly**, **Unleash**, **Flagsmith**, or **internal** flag service — evaluate in **service layer**; **document** flag keys, defaults per profile, owners, and **cleanup** after rollout.

### M9. Checklist (medium)

- [ ] All public endpoints authenticated unless explicitly public and documented.
- [ ] Migrations reviewed; rollback strategy for risky changes.
- [ ] CI: compile, tests, static analysis (SpotBugs / Checkstyle as agreed).
- [ ] **Profiles**: changes validated under **dev / staging** config before prod.
- [ ] **Observability**: dashboards or alerts cover **golden signals** for critical endpoints.
- [ ] **Feature flags**: new flags documented; safe defaults when flag service is down (if applicable).

---

## Tier: Big project

### B1. Modular monolith or services

- **Modular monolith** with **Spring Modulith** or **strict packages** before splitting microservices.
- If **microservices**: **bounded contexts**, **idempotent** consumers, **saga** / **outbox** for distributed consistency where needed.

### B2. Reliability


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onstonboy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
