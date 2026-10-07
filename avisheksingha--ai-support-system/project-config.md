---
trigger: always_on
description: This repository is a Spring Boot 4.1.1 microservices platform for AI-powered ticket management, using Spring AI with Gemini/OpenAI providers, service discovery, and event-driven workflows via Kafka.
---

# Copilot Instructions for AI Support System

This repository is a Spring Boot 4.1.1 microservices platform for AI-powered ticket management, using Spring AI with Gemini/OpenAI providers, service discovery, and event-driven workflows via Kafka.

## Build, Test, and Lint Commands

### Build

- **Build all modules:**
  ```bash
  mvn -f aisupport-parent/pom.xml clean install
  ```

- **Build a single module:**
  ```bash
  mvn -pl <module-name> clean install
  ```

- **Build without tests:**
  ```bash
  mvn -f aisupport-parent/pom.xml clean install -DskipTests
  ```

### Run Tests

- **Run all tests from repo root:**
  ```bash
  mvn test
  ```

- **Run tests for a single module:**
  ```bash
  mvn -pl <module-name> test
  ```

- **Run a specific test class:**
  ```bash
  mvn -pl <module-name> -Dtest=<TestClassName> test
  ```

- **Core test packs by service:**
  - **Ticket Service:** `mvn -pl ticket-service -Dtest=TicketControllerTest,TicketServiceBehaviorTest,GlobalExceptionHandlerTest,OutboxEventPublisherTest test`
  - **AI Analysis:** `mvn -pl ai-analysis-service -Dtest=AnalysisControllerTest,AnalysisProcessingServiceTest,AnalysisQueryServiceTest test`
  - **Routing:** `mvn -pl routing-service -Dtest=RoutingServiceTest,RuleEvaluationServiceTest test`
  - **RAG:** `mvn -pl rag-service -Dtest=RagServiceTest test`

### Run Services

- **Run a service:**
  ```bash
  cd <service-dir> && mvn spring-boot:run
  ```

- **Run with profile:**
  ```bash
  cd <service-dir> && mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=<profile>"
  ```

### Code Quality

- **SonarQube analysis:**
  ```bash
  mvn clean verify sonar:sonar
  ```

- **Generate Javadoc:**
  ```bash
  mvn javadoc:javadoc
  ```

## High-Level Architecture

- **discovery-service:** Eureka registry (Port: 8761)
- **api-gateway:** Spring Cloud Gateway entry point and correlation-id propagation (Port: 8080)
- **auth-service:** Authentication, authorization, and JWT issuance (Port: 8081)
- **ticket-service:** Ticket lifecycle, updates, and outbox publishing (Port: 8082)
- **ai-analysis-service:** Domain capability service providing sentiment, urgency, and intent analysis via Spring AI (Port: 8083)
- **routing-service:** Domain capability service for deterministic ticket routing decisions (Port: 8084)
- **rag-service:** Domain capability service providing vector embedding and contextual knowledge retrieval (Port: 8085)
- **ai-orchestration-service:** The core AI runtime. Consumes `ticket-created`, orchestrates workflows, publishes `ticket-orchestrated` (Port: 8086)
- **common-library:** Shared DTOs, enums, events, constants

### Service Startup Order

1. `discovery-service`
2. `api-gateway`
3. Core services (parallel): `auth-service`, `ticket-service`, `ai-analysis-service`, `routing-service`, `rag-service`, `ai-orchestration-service`

### API Documentation

- Services with REST controllers expose Swagger UI at `/swagger-ui.html`.
- Examples:
  - <http://localhost:8082/swagger-ui.html>
  - <http://localhost:8083/swagger-ui.html>

## Technology Stack

- **Language:** Java 21
- **Framework:** Spring Boot 4.1.1 + Spring Framework 7
- **Microservices:** Spring Cloud 2025.1.2
- **AI Integration:** Spring AI 2.0.0
- **Frontend / Runtime:** Node.js 22 + React 19 + Vite
- **Database:** PostgreSQL + PGVector
- **Messaging:** Apache Kafka
- **Service Discovery:** Netflix Eureka
- **API Documentation:** SpringDoc OpenAPI 3.0.3
- **Validation:** Jakarta Validation 3.1.1
- **Security:** Spring Security + JWT
- **Object Mapping:** MapStruct 1.6.3
- **Resilience:** Resilience4j

## Key Conventions

### Dependency Injection & Mapping

- Use constructor injection (`@RequiredArgsConstructor`) over field injection.
- Prefer explicit Lombok annotations on entities (`@Getter/@Setter`, `@NoArgsConstructor`) unless module already uses an established pattern.
- Use MapStruct with `componentModel = "spring"` where mapper beans are required.

### REST & Service Layer

- Keep controller DTOs service-specific.
- Use `@RestControllerAdvice` per service for domain-level error mapping.
- Keep transactional boundaries in service layer.
- Use `java.time.Instant` for all temporal fields — never `LocalDateTime`.
- Use `DateTimeUtil` from common-library for formatting/parsing timestamps.

### Event-Driven Communication

- Use outbox flow for cross-service event publication.
- Keep scheduler-based outbox publishers enabled where present (`@Scheduled(fixedDelay = 2000)`).
- Preserve `X-Correlation-Id` in Kafka headers and restore it into MDC in consumers.

### API Gateway Rules

- `api-gateway` is reactive (WebFlux). Do not introduce MVC stack there.
- Other services should remain servlet-based.
- External client entry point is gateway on port `8080`.

### AI & RAG

- AI analysis uses pluggable providers (`chat.provider=google-genai|openai`) via `ChatProvider`.
- RAG uses `QuestionAnswerAdvisor` + PGVector via Spring AI vector store.
- Model names and provider values come from config properties, not hardcoded literals.

### Correlation ID & Observability
- Gateway injects/passes `X-Correlation-Id`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avisheksingha/ai-support-system](https://github.com/avisheksingha/ai-support-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
