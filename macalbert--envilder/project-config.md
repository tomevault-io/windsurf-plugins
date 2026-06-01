---
trigger: always_on
description: Validates cross-provider config (profile is AWS-only,
---

# Envilder Copilot Instructions

## Project Overview

Envilder is a **multi-runtime secret management platform** that securely
centralizes environment variables from AWS SSM Parameter Store or Azure Key
Vault. The product has two halves:

1. **TypeScript core** — CLI + GitHub Action for pulling/pushing secrets (Hexagonal Architecture, InversifyJS DI)
2. **Runtime SDKs** — Independent libraries (.NET, Python, more planned)
   that load secrets directly into app processes at startup — no `.env`
   files, no intermediaries

All components share the **map-file format** as the universal contract (JSON,
Git-versioned, PR-reviewable). Built with **Clean Architecture** principles for
testability and modularity.

## Architecture Layers

### Domain Layer (`src/envilder/core/domain`)

**Pure business logic - NO external dependencies allowed.**

- **Entities**: `EnvironmentVariable` (immutable, with validation and `maskedValue` for safe logging)
- **Ports** (interfaces): `ISecretProvider`, `IVariableStore`, `ILogger`
- **Errors**: Custom domain errors (`InvalidArgumentError`, `ParameterNotFoundError`, etc.) extend `DomainError`
- **Value Objects**: `OperationMode` enum

### Application Layer (`src/envilder/core/application`)

**Use case orchestration using Command/Handler pattern.**

- Each feature = `*Command` + `*CommandHandler` pair (e.g., `PullSecretsToEnvCommand`, `PullSecretsToEnvCommandHandler`)
- `DispatchActionCommandHandler` routes between pull/push modes via switch statement
- Handlers are `@injectable()` and inject dependencies via `@inject(TYPES.X)`
- Commands have static `.create()` factory methods

### Infrastructure Layer (`src/envilder/core/infrastructure`)

**Adapters implementing domain ports.**

- `AwsSsmSecretProvider`: Implements `ISecretProvider` using `@aws-sdk/client-ssm`
- `AzureKeyVaultSecretProvider`: Implements `ISecretProvider` using `@azure/keyvault-secrets`
- `FileVariableStore`: Implements `IVariableStore` for .env and mapping JSON files (supports `$config` section)
- `ConsoleLogger`: Implements `ILogger` with colored output via `picocolors`

### Apps Layer (`src/envilder/apps`)

**Entry points (CLI and GitHub Action).**

- `cli/Cli.ts`: Uses `commander` for CLI parsing
- `gha/Gha.ts`: Reads inputs from `process.env.INPUT_*` (GitHub Actions convention)
- `shared/ContainerConfiguration.ts`: Shared DI setup (provider selection, handler binding) used by both CLI and GHA
- Each has `Startup.ts` that delegates to shared `ContainerConfiguration`

## Dependency Injection (InversifyJS)

**Symbol Registry**: `src/envilder/core/types.ts` exports `DOMAIN`, `APPLICATION`, and legacy `TYPES` objects.

**Container Setup Pattern** (see `src/envilder/apps/shared/ContainerConfiguration.ts`):

```typescript
// Provider selection via factory registry — NOT inline construction
const providerFactories: Record<string, ProviderFactory> = {
  aws: (config) => createAwsSecretProvider(config),
  azure: (config, options) => createAzureSecretProvider(config, options),
};

// In configureInfrastructureServices():
const selectedProvider = config.provider?.toLowerCase() || 'aws';
const factory = providerFactories[selectedProvider];
if (!factory) {
  throw new InvalidArgumentError(`Unsupported provider: ${config.provider}`);
}
container.bind<ISecretProvider>(TYPES.ISecretProvider)
  .toConstantValue(factory(config, options));
```

**Provider Configuration**: CLI reads `$config` from the map file and merges
with CLI flags (`--provider`, `--vault-url`, `--profile`) into a `MapFileConfig`
object passed to `configureInfrastructureServices()`. CLI flags override
`$config` values.

## Key Workflows & Commands

**Package Manager**: `pnpm` (monorepo via `pnpm-workspace.yaml`)

**Development**:

- `pnpm build` — TypeScript compilation
- `pnpm build:gha` — Bundle GitHub Action with `@vercel/ncc` into `github-action/dist/index.js`
- `pnpm verify:gha` — Verify GHA bundle is up-to-date (fails if dist is stale)
- `pnpm local:install` — Build + pack + install globally for local testing
- `pnpm local:test-run` — Run CLI against `e2e/sample/envilder.json`

**Quality**:

- `pnpm test` — Vitest with coverage (v8 provider, outputs to `coverage/`)
- `pnpm test:ci` — Adds JUnit reporter for CI pipelines
- `pnpm lint` — Runs Secretlint (credential detection), Biome (format/lint), and `tsc --noEmit`
- `pnpm format` — Auto-format with Biome

**E2E Tests**: Located in `e2e/`, use real AWS SSM via LocalStack and Azure Key Vault
via Lowkey Vault (both via TestContainers). Run `pnpm build` + `pack-and-install.ts` before E2E.

## Coding Conventions

### Command/Handler Pattern

1. **Command class**: Data container with validation via static `.create()` method
2. **Handler class**: Decorated with `@injectable()`, injects ports via constructor
3. **Registration**: Add symbol to `TYPES`, bind in `configureApplicationServices()` (`src/envilder/apps/shared/ContainerConfiguration.ts`)
4. **Routing**: Add case to `DispatchActionCommandHandler.handleCommand()` switch

Example (PushSingle):

- Command: `PushSingleCommand.create(key, value, ssmPath)`
- Handler: `PushSingleCommandHandler` injects `ISecretProvider` and `ILogger`
- Dispatched via `OperationMode.PUSH_SINGLE` case

### Error Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [macalbert/envilder](https://github.com/macalbert/envilder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
