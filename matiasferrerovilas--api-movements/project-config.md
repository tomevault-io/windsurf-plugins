---
trigger: always_on
description: Backend de gestión de finanzas personales. Permite registrar movimientos, suscripciones, ingresos y cuentas compartidas con actualizaciones en tiempo real vía WebSocket.
---

# api-movements — AGENTS.md

Backend de gestión de finanzas personales. Permite registrar movimientos, suscripciones, ingresos y cuentas compartidas con actualizaciones en tiempo real vía WebSocket.

> **Producción:** `https://movement.eva-core.com`

---

## Tech Stack

| Área | Tecnología |
|---|---|
| Language | Java 25 |
| Framework | Spring Boot 4.0.2 / Gradle 9 |
| Database | MySQL 8 + Liquibase (`ddl-auto: none`) |
| ORM | Spring Data JPA / Hibernate |
| Auth | Keycloak — OAuth2 Resource Server, JWT RS256 |
| Messaging | RabbitMQ (AMQP) + WebSocket STOMP/SockJS |
| Mapping | MapStruct 1.6.3 (`componentModel = "spring"`) |
| Boilerplate | Lombok (`@Data`, `@Builder`, `@RequiredArgsConstructor`) |
| Cache | Caffeine (in-memory, 5h TTL para currency) |
| PDF parsing | Apache PDFBox 3.0.6 |
| API docs | SpringDoc OpenAPI 3 (`/docs`) |
| Testing | Spock 2.4 + Testcontainers (MySQL) + Mockito |
| CI/CD | GitHub Actions → Docker Hub `mferrerovilas/api-movements` (ARM64) |

---

## Regla de Oro: Tests y Checkstyle Obligatorios

> **Todo cambio en el código de producción debe ir acompañado de la ejecución de tests y checkstyle antes de considerarse completo.**

```bash
# Ejecutar tests
./gradlew test

# Ejecutar checkstyle
./gradlew checkstyleMain checkstyleTest

# O ambos juntos
./gradlew test checkstyleMain checkstyleTest
```

- **No se acepta código nuevo sin cobertura** de los casos principales.
- **No se acepta código que rompa tests existentes** sin justificación explícita.
- **No se acepta código que viole el checkstyle.** Si la regla es incorrecta, se discute antes de saltearla.

---

## Package Structure

```
api.m2.movements
├── controller/         REST controllers — uno por dominio, todos bajo /v1/*
├── entities/           JPA entities (Lombok @Data + @Builder)
├── enums/              Enums de dominio: WorkspaceRole, CategoryEnum, EventType, etc.
├── exceptions/         BusinessException, EntityNotFoundException, PermissionDeniedException
├── helpers/            PDF parsers: PdfExtractprHelper (interface) + BBVA/Galicia impls + ParserRegistry
├── mappers/            MapStruct interfaces, 13 mappers, se componen entre sí
├── projections/        JPA interface projections (read-only, para queries livianas)
├── records/            DTOs como Java records, organizados por dominio
│   ├── workspaces/
│   ├── balance/
│   ├── invite/
│   ├── movements/
│   ├── users/          UserBaseRecord, UserMeRecord
│   └── ...
├── repositories/       Spring Data JPA — todos extienden JpaRepository
├── security/           JwtAuthenticationConverter + SecurityConfiguration
└── services/           Lógica de negocio, organizada por dominio
    ├── balance/
    ├── workspaces/     WorkspaceQueryService (reads) + WorkspaceAddService (writes) + MembershipService
    ├── movements/      MovementAddService + MovementGetService + MovementFactory + file import strategies
    ├── publishing/
    │   ├── rabbit/     RabbitSocketMessageService (base) + MovementPublishServiceRabbit
    │   └── websockets/ WebSocketMessageService (base) + Movement/Workspace/ServicePublishServiceWebSocket
    └── user/           UserService + UserAddService
```

---

## Entidades y Relaciones Clave

| Entity | Campos relevantes | Relaciones |
|---|---|---|
| `User` | `id: Long` (PK auto-increment), `email`, `isFirstLogin`, `userType` | base de todo |
| `Workspace` | `id`, `name` | `owner → User`, `members → WorkspaceMember[]` |
| `WorkspaceMember` | `role: WorkspaceRole` | `user → User`, `workspace → Workspace` |
| `WorkspaceInvitation` | `status: InvitationStatus` | `user → User` (invitado), `invitedBy → User`, `workspace → Workspace` |
| `Movement` | `amount`, `date`, `type`, `description`, `cuotaActual/Total` | `owner → User`, `workspace → Workspace`, `category`, `currency`, `bank` |
| `Income` | `amount` | `user → User`, `bank → Bank`, `currency`, `workspace → Workspace` |
| `Subscription` | `description`, `amount`, `lastPayment`, `@Transient isPaid()` | `owner → User`, `workspace → Workspace`, `currency` |
| `UserBank` | `createdAt` | `user → User`, `bank → Bank` |
| `UserSetting` | `settingKey: UserSettingKey`, `settingValue: Long` | `user → User` |

> **IMPORTANTE:** `User.id` es un `Long` auto-incremental de DB. El Keycloak subject (`sub` del JWT) es un UUID `String` separado. **No son intercambiables.**

---

## API Endpoints

### Usuarios y Onboarding
| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/v1/users/me` | Retorna `UserMeRecord`. Si es nuevo: `{ id: null, isFirstLogin: true }` |
| `POST` | `/v1/onboarding` | Crea usuario, cuentas e ingreso inicial |

### Movimientos
| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/v1/expenses` | Movimientos paginados con filtros |
| `POST` | `/v1/expenses` | Crear movimiento |
| `POST` | `/v1/expenses/import-file` | Importar movimientos desde PDF bancario |
| `PATCH` | `/v1/expenses/{id}` | Actualización parcial (MapStruct `IGNORE` null) |
| `DELETE` | `/v1/expenses/{id}` | Eliminar movimiento |

### Balance
| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/v1/balance` | Balance total (INGRESO/GASTO) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matiasferrerovilas/api-movements](https://github.com/matiasferrerovilas/api-movements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
