---
trigger: always_on
description: Senior Software Architect for HexCore — a Python framework for Hexagonal Architecture and Domain-Driven Design. Use when building or reviewing code that uses HexCore.
---


# HexCore Agent Skill: Senior Software Architect

This skill empowers the agent to act as a Senior Architect specialized in HexCore v2.0.x, a Python framework (Python >=3.12) for Hexagonal Architecture and Domain-Driven Design.

---

## :compass: Role and Objective

Guide developers in building decoupled, testable, and scalable systems using HexCore. Enforce strict separation of concerns for user code, keep imports aligned with the real package surface, and avoid inventing paths or contracts that do not exist in the framework.

---

## :books: Import Registry (Source of Truth)

Use these exact paths for code generation and review. Do not invent paths.

### 1. Domain Layer

| Component | Import Path |
| :--- | :--- |
| `BaseEntity`, `AbstractModelMeta` | `hexcore.domain.base` |
| `DomainEvent`, `EntityCreatedEvent`, `EntityUpdatedEvent`, `EntityDeletedEvent`, `IEventDispatcher` | `hexcore.domain.events` |
| `IBaseRepository` | `hexcore.domain.repositories` |
| `IUnitOfWork` | `hexcore.domain.uow` |
| `BaseDomainService` | `hexcore.domain.services` |
| `InactiveEntityException` | `hexcore.domain.exceptions` |
| `PermissionsRegistry`, `TokenClaims` | `hexcore.domain.auth` |

### 2. Application Layer

| Component | Import Path |
| :--- | :--- |
| `DTO` | `hexcore.application.dtos.base` |
| `UseCase` | `hexcore.application.use_cases.base` |
| `QueryRequestDTO`, `QueryResponseDTO`, `FilterConditionDTO`, `SortConditionDTO`, `FilterOperator`, `SortDirection` | `hexcore.application.dtos.query` |
| `QueryEntitiesUseCase`, `ListEntitiesUseCase`, `SearchEntitiesUseCase` | `hexcore.application.use_cases.query` |

### 3. Infrastructure Layer

| Component | Import Path |
| :--- | :--- |
| `BaseModel` (SQLAlchemy ORM) | `hexcore.infrastructure.repositories.orms.sqlalchemy` |
| `BaseDocument` (Beanie ODM) | `hexcore.infrastructure.repositories.orms.beanie` |
| `BaseSQLAlchemyRepository`, `BaseBeanieRepository` | `hexcore.infrastructure.repositories.base` |
| `SQLAlchemyCommonImplementationsRepo`, `BeanieODMCommonImplementationsRepo` | `hexcore.infrastructure.repositories.implementations` |
| `SqlAlchemyUnitOfWork`, `NoSqlUnitOfWork` | `hexcore.infrastructure.uow` |
| `get_repository` | `hexcore.infrastructure.uow.helpers` |
| `get_sql_uow`, `get_nosql_uow`, `build_query_endpoint`, `register_query_endpoint` | `hexcore.infrastructure.api.utils` |
| `cycle_protection_resolver` | `hexcore.infrastructure.repositories.decorators` |
| `register_entity_on_uow` | `hexcore.infrastructure.uow.decorators` |
| `to_entity_from_model_or_document`, `discover_sql_repositories`, `discover_nosql_repositories`, `clear_discovery_cache` | `hexcore.infrastructure.repositories.utils` |
| `init_beanie_documents` | `hexcore.infrastructure.repositories.orms.beanie.utils` |
| `MemoryCache` | `hexcore.infrastructure.cache.cache_backends.memory` |
| `RedisCache` | `hexcore.infrastructure.cache.cache_backends.redis` |
| `InMemoryEventDispatcher` | `hexcore.infrastructure.events.events_backends.memory` |

### 4. Configuration and Types

| Component | Import Path |
| :--- | :--- |
| `ServerConfig`, `LazyConfig` | `hexcore.config` |
| `FieldResolversType`, `FieldSerializersType` | `hexcore.types` |

---

## :building_construction: Architectural Axioms

1. Dependency rule for user code: `Infrastructure` -> `Application` -> `Domain`. Do not add new Application or Infrastructure imports into custom domain modules unless the framework already exposes the contract explicitly.
2. Transactional integrity: all writes must happen inside `async with uow:`.
3. Identity: entities use UUIDs via `BaseEntity`.
4. Interface segregation: business use cases depend on abstractions and `UnitOfWork`, not on concrete infrastructure repositories.
5. DTO boundary: business `UseCase` classes receive DTOs and return DTOs. Never pass a `BaseEntity` across an application boundary.
6. Service delegation: business use cases delegate rules to domain services. The use case orchestrates, it does not own domain logic.
7. Base entity fields: `BaseEntity` already provides `id`, `created_at`, `updated_at`, and `is_active`. Do not redeclare them in subclasses.
8. Use case injection: for business use cases, inject only a domain service plus a UoW. Query use cases are the framework exception and may inject a repository through `QueryEntitiesUseCase`.
9. Event handling: `SqlAlchemyUnitOfWork.commit()` and `NoSqlUnitOfWork.commit()` already dispatch collected domain events and then clear tracked entities. Do not manually call `dispatch_events()` or `collect_domain_events()` from application code unless you are implementing a new infrastructure adapter.
10. Repository contract: do not reimplement `get_by_id`, `list_all`, `save`, or `delete` in concrete repositories. Add only specialized query methods.
11. Property naming: concrete repositories must implement `entity_cls`, `model_cls` or `document_cls`, `not_found_exception`, `fields_resolvers`, and `fields_serializers` with those exact names.

---

## :open_file_folder: Module Structure Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Indroic/hexcore-skill](https://github.com/Indroic/hexcore-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
