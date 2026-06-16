---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**BankForge**

A realistic Australian core banking platform demonstrating enterprise microservices patterns — 4 Java/Spring Boot services with local ACID transactions, Saga/Outbox/CDC messaging, full observability (Jaeger + Prometheus + Loki + Grafana), an Istio service mesh on Kubernetes, and a Claude MCP server that queries the live system. Built as a learning sandbox and long-term foundation for an AI-driven root cause analysis system.

**Core Value:** A running, end-to-end system where every enterprise pattern (ACID, Saga, Outbox, mTLS, distributed tracing) is implemented and queryable via AI agent — proving the patterns work together, not just in theory.

### Constraints

- **Tech stack:** Java 21 LTS + Spring Boot 4.0.5 — locked for virtual threads and built-in OTel
- **Container runtime:** Podman (daemonless, rootless) — not Docker
- **Local only:** kind cluster, no cloud provider dependencies
- **Database isolation:** Each service owns its own PostgreSQL schema (no shared DB)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## CRITICAL FLAG: Spring Boot 4.0.5 Version Risk
- Spring Boot 3.3.x / 3.4.x were the stable production releases
- Spring Boot 4.0 was announced targeting Q4 2025 / Q1 2026, requiring Spring Framework 7.0
- Spring Framework 7.0 was itself in milestone phase as of mid-2025
- A specific patch version like `4.0.5` suggests months of post-release stabilisation that may not have happened yet
## Recommended Stack
### Core Java / Spring
| Technology | Recommended Version | Plan Version | Status | Confidence |
|------------|--------------------|-----------|----|------------|
| Java | 21 LTS | 21 LTS | CONFIRMED | HIGH |
| Spring Boot | **3.4.x or 4.0.x (verify GA)** | 4.0.5 | NEEDS VERIFICATION | LOW |
| Spring Framework | 6.2.x (if SB 3.4) / 7.0.x (if SB 4.0) | (implied by SB 4.0.5) | NEEDS VERIFICATION | LOW |
| Spring Data JPA | Bundled with Spring Boot | — | OK | HIGH |
| Spring Kafka | Bundled with Spring Boot | — | OK | HIGH |
| Spring Security | Bundled with Spring Boot | — | OK | HIGH |
- Micronaut and Quarkus both compete on cold-start time (relevant for Lambda/FaaS, not relevant for long-running K8s services)
- Spring Boot has the most mature Debezium + Outbox + Saga tooling in the Java ecosystem
- Spring Boot 4.0 / 3.4 both ship OTel auto-configuration out of the box — the plan's key "no manual TracerProvider" requirement is met by both
- Quarkus is worth knowing but switching mid-project from Spring Boot would be a significant cost with no banking-relevant benefit
- If 4.0.x is GA and has been out for 2+ months: use 4.0.x (Spring Framework 7, virtual threads first-class, better OTel integration)
- If 4.0.x is still RC/milestone: use 3.4.x — every feature in the plan is available there
- Do NOT use a milestone or RC for the foundation of a banking platform learning project; debugging framework bugs on top of pattern complexity is counterproductive
### Application Frameworks & Libraries
| Library | Recommended Version | Purpose | Why |
|---------|-------------------|---------|-----|
| Spring Data JPA | (bundled with SB) | ORM / repository layer | Standard; HibernateJPA integration is best-in-class |
| Spring Kafka | (bundled with SB) | Kafka consumer/producer | Native Spring idioms, @KafkaListener, error handling |
| Spring State Machine | 4.0.x | Transfer state machine | Purpose-built FSM for Spring; avoids rolling own state machine |
| Resilience4j | 2.2.x | Circuit breaker (service-to-service) | Spring Boot native integration; replaces deprecated Hystrix |
| Flyway | 10.x | DB migration | Repeatable schema migrations; critical for outbox table setup |
| MapStruct | 1.6.x | DTO ↔ entity mapping | Compile-time, zero-reflection mapping; better than ModelMapper |
| Lombok | 1.18.x | Boilerplate reduction | Use judiciously; @Builder and @Value are safe bets |
| jackson-databind | (bundled with SB) | JSON serialisation | Default; do not replace with Gson |
### Database
| Technology | Recommended Version | Plan Version | Status | Confidence |
|------------|--------------------|-----------|----|------------|
| PostgreSQL | 16 or 17 | 15 | UPDATE RECOMMENDED | MEDIUM |
| Redis | 7.2.x | 7 | OK (pin to 7.2) | HIGH |
| Neo4j | 5.x Community | 5 | OK | HIGH |
- PostgreSQL 15 is still receiving security updates but entered "minor releases only" mode in Q4 2024
- PostgreSQL 16 added parallel query improvements and logical replication improvements (relevant for Debezium)
- PostgreSQL 17 (released September 2024) is the current major version with further logical replication refinements
- **Recommendation: Use PostgreSQL 16 or 17.** The Debezium CDC connector benefits from PostgreSQL 16+'s improved logical replication slot stability. Using 15 is not wrong but misses small Debezium reliability improvements.
- The outbox table design in plan.md is compatible with all three versions.
### Messaging & Event Streaming
| Technology | Recommended Version | Plan Version | Status | Confidence |
|------------|--------------------|-----------|----|------------|
| Apache Kafka | 3.8.x or 3.9.x | unspecified | SPECIFY VERSION | HIGH |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Harry-Zhao-AU/BankForge](https://github.com/Harry-Zhao-AU/BankForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
