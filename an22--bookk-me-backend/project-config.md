---
trigger: always_on
description: Modular monolith split into Ktor microservices. Kotlin + Ktor (CIO) + Koin DI + Exposed ORM + kotlinx ProtoBuf serialization + MockK/JUnit5 tests. Follow the recipes below verbatim; they mirror the real code (reference implementation: `service/appointments`, the newest service).
---

# bookk-server — Agent Playbook

Modular monolith split into Ktor microservices. Kotlin + Ktor (CIO) + Koin DI + Exposed ORM + kotlinx ProtoBuf serialization + MockK/JUnit5 tests. Follow the recipes below verbatim; they mirror the real code (reference implementation: `service/appointments`, the newest service).

## Module map

```
core/                    shared infra: domain Result/Error types, service (Ktor server, auth, respondWith), data (Exposed, event streaming, cache)
library/                 money, idempotency, permissions
service/<svc>/
  domain/api/            operation interfaces + entities + <Svc>ErrorCodes   (package com.bookk.<svc>.domain.api.{operation,entity})
  domain/impl/           <Operation>Impl + di/DI.kt + tests                  (package com.bookk.<svc>.domain.impl.{operation,di})
  data/source/           datasource INTERFACES                               (package com.bookk.<svc>.domain.datasource)
  data/                  datasource impls, orm tables/entities, migration, di
  microservice/          route/<Svc>Routing.kt (typed Resources), route/api/*Route.kt, <Svc>Microservice.kt main, route tests
  client/                cross-service events/API
```

New gradle modules must be registered in `settings.gradle.kts` (one `include` per submodule, grouped per service). Convention plugins (`libs.plugins.bookk.microservice`, `bookk.domain.impl`, `bookk.domain.api`, `bookk.data`, …) already add Ktor/Koin/MockK/kotlin-test deps and JUnit platform — do not re-add them.

**Never generate database migrations.** When an ORM table changes (new column, new index, etc.), leave the table definition updated but do NOT bump `referenceVersion`/`targetVersion` in `<Svc>Migration.kt` or run its `main()` to produce a new `V<n>__migration_script.sql`. The user runs that generation step themselves.
**Ignore all git operations.** Managing Git is the responsibility of the developer, do not automatically commit, push or merge

## Core conventions (apply everywhere)

- **Never write comments.** No `//` comments, no KDoc, no explanatory blocks — in production code or tests. Express intent through names and structure instead: rename the function, extract the condition into a named value, split the method. If something seems to need a comment, that is a signal the code should be clearer. The only exceptions are the route KDoc the openApi plugin parses (`Summary:`, `Body:`, `Response:`, …) and comments that already exist in the file — never delete or reword someone else's comment.
- Operations return `Result<T>`; impls **throw** errors inside `transactionManager.transaction { }`, which catches and converts to `Result.failure`.
- Domain errors: nested `sealed interface Error` in the operation interface; each case is a `class` extending `BusinessError(statusCode, code, message)` (classes, NOT data objects). Assert with `is`, never equality.
- Error codes live in `domain/api/.../<Svc>ErrorCodes` as `BASE + n`. Blocks: auth=0, user=100000, business=200000, appointments=300000. Next service takes the next 100000 block.
- Generic infrastructure errors: `com.bookk.core.domain.entity.Error` (`NotFound`, `OperationNotAllowed`, …).
- `call.respondWith(result)` (core/service) maps: success Unit→204, success T→200, `BusinessError`→its statusCode + `SimpleServerError(errorCode, message)`, `Error.NotFound`/`Error.OperationNotAllowed`→**404** (intentional: permission failures do NOT return 403), anything else→500 (logged).
- Permissions: `permissionsDataSource.getPermissions(userId, businessId).assert(ObjectPermission.EDIT)` (library/permissions) — throws `Error.OperationNotAllowed`.
- Wire format is ProtoBuf (`application/x-protobuf`) for all bodies/responses. **A nullable collection (`List<T>?`, `Map<K, V>?`) cannot be serialized when null** — kotlinx throws `'null' is not supported as the value of collection types in ProtoBuf`. For an optional group of fields in a partial-update DTO, wrap them in a nullable `@Serializable` holder class (a nullable message is fine) instead of making each list nullable — see `BusinessUpdateModel.schedule: Schedule?`. **A nullable property must not also have a default** — the serializer runs with `encodeDefaults = true`, and encoding a defaulted null throws `'null' is not supported for optional properties in ProtoBuf` as soon as a caller omits it. Give every nullable field on a partial-update DTO no default at all and pass them explicitly (`BusinessUpdateModel`, `UserEditModel`).
- Entities: `@Serializable data class` in `domain/api/.../entity` with a `companion object { fun stub(...) }` factory (defaulted params, `Uuid.random()`, `Instant.fromEpochMilliseconds(0)`) — add `stub()` to every new entity; tests rely on it.

## Recipe: new business operation

Files to touch (example names from appointments):

1. `domain/api/.../operation/DoThing.kt` — interface + errors:
```kotlin
interface DoThing {
    suspend operator fun invoke(userId: Uuid, ...): Result<Thing>

    sealed interface Error {
        class ThingExists : BusinessError(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [an22/bookk.me-backend](https://github.com/an22/bookk.me-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
