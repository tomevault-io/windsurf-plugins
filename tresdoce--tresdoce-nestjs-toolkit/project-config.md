---
trigger: always_on
description: This file is machine-readable context for AI assistants working in this codebase. It is meant to be loaded once at the start of a session so the assistant does not need to re-read every file.
---

# tresdoce-nestjs-toolkit — LLM Context

This file is machine-readable context for AI assistants working in this codebase. It is meant to be loaded once at the start of a session so the assistant does not need to re-read every file.

---

## 1. Project overview

A NestJS monorepo that publishes ~22 independent packages to npm under the `@tresdoce-nestjs-toolkit` scope. All packages are designed to be consumed by [nestjs-starter](https://github.com/rudemex/nestjs-starter) or any project that follows its centralized-config architecture.

| Dimension              | Detail                                                                    |
| ---------------------- | ------------------------------------------------------------------------- |
| Runtime                | Node.js v22.21.1                                                          |
| Framework              | NestJS 11.x (`@nestjs/common`, `@nestjs/core`, etc.)                      |
| Language               | TypeScript 6.x                                                            |
| Package manager        | Yarn 1.22.22 (classic workspaces)                                         |
| Monorepo orchestration | Lerna 9 + Turbo 2                                                         |
| Build tool             | `@pika/pack` (NOT `tsc` directly)                                         |
| Commit style           | Conventional Commits (`feat`, `fix`, `chore`, `docs`, `refactor`, `test`) |

Packages live under `packages/<name>/src/`. Published artifacts are placed at `dist/<name>/` from the repo root.

---

## 2. Centralized config pattern (most critical)

Every module in this toolkit reads its configuration from a single `ConfigService` key: `'config'`. The shape of that key is `Typings.AppConfig` defined in `packages/core/src/typings/index.ts`.

### How the host application registers config

```typescript
// In the consuming app (e.g., nestjs-starter)
import { registerAs } from '@nestjs/config';
import { Typings } from '@tresdoce-nestjs-toolkit/core';

export default registerAs('config', (): Typings.AppConfig => ({
  project: { ... },
  server: { ... },
  swagger: { ... },
  redis: { ... },   // optional — RedisModule activates only if present
  // etc.
}));
```

`ConfigModule` must be imported with `isGlobal: true` in the host `AppModule`.

### AppConfig shape (abbreviated)

```typescript
// packages/core/src/typings/index.ts
export interface AppConfig {
  project: IProjectConfig; // required
  server: IServerConfig; // required
  swagger: ISwaggerConfig; // required
  health?: IHealthConfig; // optional
  params?: IParamsConfig; // optional
  httpClient?: IHttpClientConfig; // optional → HttpClientModule
  services?: Record<string, IServicesConfig>; // optional
  database?: DatabaseOptions; // optional → TypeOrmClientModule (key: 'config.database.typeorm')
  redis?: RedisOptions; // optional → RedisModule
  mailer?: MailerOptions; // optional → MailerModule
  camunda?: CamundaOptions; // optional → CamundaModule
  elasticsearch?: ElasticsearchOptions; // optional → ElkModule
  tracing?: TracingOptions; // optional → TracingModule
  redact?: RedactOptions; // optional → RedactModule
  bcrypt?: BcryptOptions; // optional → BcryptModule
  snowflakeUID?: SnowFlakeOptions; // optional → SnowFlakeModule
  sqs?: AwsSqsModuleOptions; // optional → AwsSqsModule
  [key: string]: any;
}
```

### Module activation rule

A module that reads an optional config key (e.g., `redis`) is **only** active when that key is populated in the config object. There is no runtime guard in the modules themselves; if the key is absent `ConfigService.get('config.redis')` returns `undefined`, and the module either fails or no-ops depending on its provider factory.

### Example: how modules read from ConfigService

```typescript
// Typical pattern in any package module
{
  provide: MODULE_OPTIONS_TOKEN,
  useFactory: async (configService: ConfigService) =>
    configService.get<FeatureOptions>('config.<featureKey>'),
  inject: [ConfigService],
}
```

The `health` and `archetype` modules read the entire `AppConfig` object:

```typescript
configService.get<Typings.AppConfig>('config');
```

---

## 3. Package dependency graph

Internal dependencies only (excludes NestJS peer deps and third-party libs):

```
core         ← base (no internal deps)
utils        ← base (no internal deps; contains RedactModule, FormatModule, BcryptModule)
filters      ← core
tracing      ← utils (FormatService)
elk          ← utils (RedactModule, FormatService, RedactService)
archetype    ← core (Typings.AppConfig)
health       ← core (Typings), uses @nestjs/terminus (does NOT add internal deps)
http-client  ← core (Typings.IHttpClientConfig)
response-parser ← (no internal deps)
rate-limit   ← (no internal deps; re-exports @nestjs/throttler)
redis        ← (no internal deps)
typeorm      ← (no internal deps; wraps @nestjs/typeorm)
paas         ← core, filters, health, rate-limit, response-parser, tracing, utils

Standalone (no internal deps):
  aws-sqs, camunda, commons, dynamoose, mailer, qrcode, snowflake-uid, test-utils
```

Key facts:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tresdoce/tresdoce-nestjs-toolkit](https://github.com/tresdoce/tresdoce-nestjs-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
