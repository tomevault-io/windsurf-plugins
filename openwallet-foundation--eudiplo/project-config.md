---
trigger: always_on
description: The backend is migrating toward pragmatic hexagonal / ports-and-adapters boundaries. The canonical [target architecture](../../apps/docs/docs/contributing/backend-architecture.md) defines roles and migration scope; the [backlog](../../apps/backend/docs/refactoring-plan.md) is not authorization to execute work.
---


# EUDIPLO Backend Architecture Instructions

The backend is migrating toward pragmatic hexagonal / ports-and-adapters boundaries. The canonical [target architecture](../../apps/docs/docs/contributing/backend-architecture.md) defines roles and migration scope; the [backlog](../../apps/backend/docs/refactoring-plan.md) is not authorization to execute work.

Apply these rules to new or explicitly migrated application/domain code. Do not classify all existing services as application code or refactor unrelated areas. Preserve external behavior unless the requested task explicitly changes it. Minimal NestJS DI decorators are allowed on application classes that can be constructed directly with fake ports; domain logic remains framework-independent.

## Architectural model

The preferred dependency direction is:

```text
Inbound adapter → Application / use case → Domain
                          ↓
                    Outbound port
                          ↑
                 Infrastructure adapter
```

The backend should remain capability-oriented. Top-level modules such as `issuer`, `verifier`, `session`, `trust`, `crypto`, `registrar`, `auth`, and `storage` may keep their feature-oriented structure.

Do not reorganize the whole backend into artificial global `domain/`, `application/`, and `infrastructure/` folders unless there is a clear benefit.

## Forbidden dependencies in application/domain code

Application/domain code must not directly depend on:

- `express`, `axios`, `undici`, `node:http(s)`, `node:fs`
- `typeorm` and any `@nestjs/*` package except `@nestjs/common`
- from `@nestjs/common`, anything except `Inject`, `Injectable`, `Optional` (so no HTTP exceptions, no `Logger`)
- `nestjs-otel`, `@opentelemetry/*`
- AWS/Azure SDK clients, Vault clients, Keycloak admin clients
- `class-validator`, `class-transformer`, `nestjs-zod`, `nestjs-pino` (plain `zod` schemas are fine)
- other infrastructure-specific SDKs

These dependencies belong in adapters or composition/bootstrap code. The authoritative list is `forbiddenPackages` in `apps/backend/test/architecture/dependency-rules.ts`.

Code placement, error mapping and DI wiring rules: see "Feature folder shape" in the target architecture. In short: `adapters/` holds only port implementations (never park orchestration there), missing resources extend `NotFoundError` from `shared/domain/not-found-error.ts`, framework-free classes with dependencies are registered via `useFactory`, and services receive plain values instead of the Express `Request`.

## Controllers

Controllers are inbound adapters.

Controllers may handle:

- request parsing
- response shaping
- protocol/HTTP headers
- HTTP status codes
- `WWW-Authenticate`
- raw HTTP body handling where required
- mapping application errors to protocol errors

Controllers should not:

- access TypeORM repositories
- contain business logic
- perform remote integration logic
- select infrastructure implementations

## Application services and use cases

Application services should describe EUDIPLO behaviour.

Examples:

- `ProcessCredentialRequest`
- `CreateCredentialOffer`
- `HandleCredentialNotification`
- `CreatePresentationRequest`
- `ProcessPresentationResponse`
- `ResolveCredentialClaims`

Use cases should depend on explicit ports where they need infrastructure.

## Ports

Ports should represent meaningful EUDIPLO capabilities.

Good examples:

- `SessionRepository`
- `CredentialClaimsProvider`
- `CredentialNotificationPublisher`
- `PresentationResultPublisher`
- `FederationResolver`
- `TrustListProvider`
- `CredentialIssuerFormat`
- `CredentialVerifierFormat`

Avoid generic application abstractions such as:

- generic `HttpClient`
- generic `Repository<T>`
- generic wrappers around TypeORM or Axios

## Persistence

TypeORM belongs behind persistence adapters.

Map persistence entities to application/domain models where doing so prevents persistence concerns from leaking into application APIs.

Do not return TypeORM entities from application-facing ports by default.

## Sessions

Session persistence should be accessed through a `SessionRepository` port.

Scheduling, metrics, persistence, event publication, and lifecycle/business behaviour should remain separable concerns.

## OID4VCI

OID4VCI orchestration must not directly depend on:

- Express requests
- TypeORM repositories
- webhook HTTP implementation details
- credential format implementation details

Break large flows into focused use cases.

OID4VCI should orchestrate credential issuance, not implement credential formats.

## Credential formats

Credential formats are first-class extension points.

At minimum, SD-JWT VC and mdoc should be isolated behind explicit contracts.

Format-specific logic should remain inside the respective implementation where possible.

Avoid format checks spread throughout orchestration logic.

Prefer:

```text
OID4VCI / OID4VP
      ↓
Credential format port
      ↓
SD-JWT VC / mdoc adapter
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwallet-foundation/eudiplo](https://github.com/openwallet-foundation/eudiplo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
