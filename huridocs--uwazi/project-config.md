---
trigger: always_on
description: Uwazi is a flexible database application to capture and organise collections of information with a particular focus on document management. HURIDOCS started Uwazi and is supporting dozens of human rights organisations globally to use the tool.
---

# Agents context

## Project Overview

Uwazi is a flexible database application to capture and organise collections of information with a particular focus on document management. HURIDOCS started Uwazi and is supporting dozens of human rights organisations globally to use the tool.

## Backend

### Tech Stack

Backend is in `app/api`

- NodeJS
- Express

### Commands

- **Install:** `yarn install`
- **Test:** `yarn test app/api`
- **Suggested Jest invocation for targeted test runs:** `DEBUG=true node --no-experimental-fetch ./node_modules/.bin/jest <path-or-pattern> -w=4`
- **Run:** `yarn hot`
- **Translations CSV update:** never edit translation keys manually in CSV files; run `yarn update-translations-csv` instead.

### Architecture Status

The backend is currently a mix of V1 (legacy) and V2 (new core) code. The migration is in progress.

#### V1

- Legacy, CRUD-centric architecture with no clear separation of concerns.
- Business rules, persistence logic, and HTTP concerns are mixed in the same modules.
- Batch operations trigger single sequential updates, causing performance bottlenecks.
- Example: `app/api/entities`

#### V2

- Located in `app/api/core`
- Hexagonal/DDD architecture: business logic in a clean core, persistence and delivery as adapters.
- The core covers the basic data model: entities, templates, files, thesauri, relationships.
- Some V2 modules still depend on the legacy layer via `app/api/core/v1_layer/` — this is intentional and temporary during the transition.

### Design Principles

- **Hexagonal architecture** — domain logic is independent of persistence and delivery mechanisms.
- **CQRS** — use cases as commands; queries as services or simple controller implementations depending on complexity.
- **`core` must not depend on external modules** — only on itself.
- **Domain models encapsulate business rules** — validation, state changes, and invariants live in the domain layer, not in services or controllers.
- **Use cases as clear entry points** — each use case represents a complete application action. Reading a single use case file should tell the full story of that action. Use cases may trigger async jobs when operations are too expensive to handle synchronously.
- **Adapters bridge domain and infrastructure** — repositories (DataSources) hide persistence details; controllers, jobs, and CLI scripts act as delivery mechanisms without leaking infrastructure concerns.
- **Contracts define external boundaries** — schemas and TypeScript types define the interface between the application layer and the outside world.

### Current `app/api/core` Structure

```
app/api/core/
 ├─ domain/                   ← domain models (Entity, Template, Thesaurus, File, ...)
 ├─ application/
 │    ├─ contracts/           ← repository/service interfaces
 │    └─ ...                  ← use cases and application services
 ├─ infrastructure/
 │    ├─ mongodb/             ← repository implementations
 │    ├─ elasticSearch/       ← search adapter
 │    ├─ express/             ← HTTP controllers
 │    ├─ jobs/                ← async job implementations
 │    ├─ factories/           ← dependency wiring
 │    └─ ...
 ├─ libs/                     ← internal shared libraries (queue, logger, eventEmitter, shell, ...)
 ├─ testing/                  ← shared test helpers
 └─ v1_layer/                 ← temporary bridge to legacy code during migration
```

### Building Blocks

Explains main backend core artifacts: purpose, practices, and testing approaches.

#### ExecutionContext

- **Purpose**: Holds shared dependencies, tenant, and actor (future: targetLanguage).
- **Access Rule**: Only usable within factories. No direct access from other layers.

#### Factories

- **Purpose**: Standardize creation of core objects with dependency wiring.
- **Signature**: Accept optional override object matching the exact dependencies of the object being built.
- **Defaults**: Use shared `ExecutionContext` dependencies (including actor/tenant) by default.
- **Testing**: Use `testingEnvironment.runWithContext(() => Factory.default())` instead of mocking.
- **Examples**: api/core/infrastructure/factories/DeleteTemplateUseCaseFactory.ts, api/core/infrastructure/factories/EntitiesServiceFactory.ts

#### UseCases

- **Purpose**: Entry points to the application, orchestrating full application flows.
- **Structure**: Extend AbstractUseCase<Input, Output, Deps> from core/libs/UseCase.ts:
  - **Input**: Defines input type (may include Zod validation schema, e.g., MultiUpdateEntity.InputSchema)
  - **Output**: Return type, usually domain models (e.g., Entity) or arrays of domain models
  - **Deps**: Lists dependencies matching contracts in application/contracts/ (e.g., TransactionManager, EntitiesService, data sources)
- **Provided by AbstractUseCase**:
  - Access to actor (this.actor/this.actorId), tenant (this.tenant), target language (this.targetLanguage) from ExecutionContext (wired via factory)
  - Shared dependencies: this.transactionManager, this.idGenerator, this.dispatcher, this.eventBus, this.logger

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huridocs/uwazi](https://github.com/huridocs/uwazi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
