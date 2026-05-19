---
trigger: always_on
description: - **Monorepo**: Contains multiple apps (backend, client, webhook) and shared packages.
---

# Copilot Instructions for EUDIPLO

## Project Architecture
- **Monorepo**: Contains multiple apps (backend, client, webhook) and shared packages.
- **Backend**: [apps/backend](../apps/backend) — NestJS API server, main business logic, protocol abstraction.
- **Client**: [apps/client](../apps/client) — Angular web UI for managing credentials, keys, and sessions.
- **Webhook**: [apps/webhook](../apps/webhook) — Cloudflare Worker webhook simulator.
- **Packages**: [packages/eudiplo-sdk-core](../packages/eudiplo-sdk-core) — Shared SDK core library.
- **Deployment**: [deployment/](../deployment) — Docker Compose configs for minimal/full setups. See [deployment/README.md](../deployment/README.md).
- **Monitoring**: [monitor/](../monitor) — OpenTelemetry Collector, Prometheus, Tempo, Loki & Grafana for observability.

## Developer Workflows
- **Install dependencies**: `pnpm install` (root)
- **Build all**: `pnpm build`
- **Start backend**: `pnpm --filter @eudiplo/backend dev` or use Docker Compose
- **Start client**: `pnpm --filter @eudiplo/client start`
- **Run all via Docker Compose**: `docker compose up -d` (see [deployment/README.md](../deployment/README.md))
- **Testing**: Use `pnpm test` or framework-specific commands in each app
- **Generate API types**: `pnpm run gen:api` (from root)

## Patterns & Conventions
- **API Design**: RESTful, protocol-agnostic endpoints. See [apps/backend/src](../apps/backend/src).
- **NestJS Modules**: Each feature has its own module with controllers, services, and subfolders:
  - `dto/` — Data Transfer Objects (request/response classes)
  - `entities/` — Database entities (TypeORM)
  - `exceptions/` — Custom exceptions
- **DTOs**: Always place DTOs in a `dto/` folder within the module. Never define DTOs inline in controllers or services. This keeps controllers and services clean and focused on their responsibilities.
- **Credential Configs**: JSON-based, managed via client UI and backend API.
- **Key Management**: Pluggable, supports filesystem and cloud KMS (see backend config).
- **Session Management**: Real-time updates via polling in client.
- **Environment Variables**: Each app has its own `.env` or `example.env`.
- **Testing**: Use framework-native tools (Vitest for backend, Angular CLI for client).
- **Docs**: Main docs in [docs/](../docs), API docs via Compodoc.

## Code Style & Quality
- Follow `tsconfig.base.json` strict settings. Prefer ES2022+ features (async/await, optional chaining, class fields).
- Use **Dependency Injection** everywhere in NestJS — never instantiate services manually.
- Use **`@InjectRepository`** for TypeORM repositories — never use `getRepository` helpers.
- Never return raw entities from controllers — always map to DTOs.
- Use **class-validator** for input validation (primary validation library in this project).
- Prefer **Composition over Inheritance** for features and providers.

## Rules for Backend Code
- When creating a module, always generate `<feature>.module.ts`, `<feature>.controller.ts`, `<feature>.service.ts` and create subfolders: `dto/`, `entities/`, `exceptions/` as needed.
- Always add Swagger annotations (`@ApiTags`, `@ApiOperation`, `@ApiResponse`, `@ApiBody`) on all controller endpoints.
- Use the **Pino logger** (`nestjs-pino` / `PinoLogger`). For audit logging (compliance events persisted to DB), use `AuditLogService`.
- Always wrap external calls in `try/catch` and throw domain-specific exceptions from the module's `exceptions/` folder.
- Custom exceptions must extend NestJS `HttpException` — there is no custom base exception class.
- When adding credential/protocol-related functions, follow existing abstractions in `packages/eudiplo-sdk-core`. Never duplicate protocol logic across modules.
- Protocol logic lives in feature modules: OID4VCI in `issuer/issuance/oid4vci/`, OID4VP in `verifier/oid4vp/`.

## Rules for Angular Client
- All forms must use **Reactive Forms** — never template-driven forms.
- API requests must use the **generated API client** from `@eudiplo/sdk-core` (`pnpm run gen:api`) — no inline HTTP URLs.
- Use **standalone components** unless a module is specifically required.
- Follow **Smart/Dumb component** pattern:
  - Smart components: orchestrate data and logic
  - Dumb components: only receive `@Input` / emit `@Output` (UI only)
- Store shared state in services with `BehaviorSubject` for state and `Observable` for consumption.

## Protocol Flow Rules (OID4VCI / OID4VP)
- **Credential Issuance**: Always validate Access Tokens and DPoP (if enabled) before issuing. Never hardcode client metadata; use resolved metadata from configuration.
- **Presentation Flows**: Use nonce endpoints and replay prevention correctly. Always verify Wallet Attestation before mapping user data (if `walletAttestationRequired` is configured).
- Follow existing flow patterns in the feature module — never duplicate protocol logic.

## Database & Migrations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwallet-foundation/eudiplo](https://github.com/openwallet-foundation/eudiplo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
