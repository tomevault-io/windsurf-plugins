---
trigger: always_on
description: **axe-api** is a TypeScript-first Node.js framework for building REST APIs quickly. It works on top of Express.js and Knex (SQL query builder), using a **model-driven, convention-over-configuration** approach where declaring a `Model` class is enough to automatically expose paginate, show, insert, update, patch, and delete HTTP endpoints.
---

# GitHub Copilot Instructions — axe-api

## Project overview

**axe-api** is a TypeScript-first Node.js framework for building REST APIs quickly. It works on top of Express.js and Knex (SQL query builder), using a **model-driven, convention-over-configuration** approach where declaring a `Model` class is enough to automatically expose paginate, show, insert, update, patch, and delete HTTP endpoints.

This repository is a **Lerna monorepo** with the following packages:

| Package           | Path                     | Purpose                           |
| ----------------- | ------------------------ | --------------------------------- |
| `axe-api`         | `packages/axe-api/`      | Core framework library            |
| `axe-api-dev-kit` | `packages/dev-kit/`      | Local development sandbox app     |
| `integrations`    | `packages/integrations/` | Integration test suite (multi-DB) |
| `docs`            | `packages/docs/`         | VitePress documentation site      |

---

## Tech stack

- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js
- **HTTP layer**: Express.js
- **Database**: Knex query builder + knex-schema-inspector (supports MySQL, MariaDB, PostgreSQL, SQLite, CockroachDB)
- **Test runner**: Vitest
- **Validation**: `validatorjs` or `robust-validator` (configurable)
- **Search**: Elasticsearch (`@elastic/elasticsearch`)
- **Caching/Rate-limit**: Redis or in-memory adaptor
- **DI container**: Custom `IoCService` (static, string-keyed)
- **Env config**: `dotenv`

---

## Repository layout (`packages/axe-api/src/`)

```
Builders/        ModelTreeBuilder, RouterBuilder, IndexBuilder
Exceptions/      Custom AxeError types
Handlers/        Express route handler entry points (RequestHandler, SwaggerHandler, …)
Middlewares/     Express middlewares (rate limit, etc.)
Phases/          Per-handler pipeline phases (query, hook, event, validation, …)
Resolvers/       ModelResolver, VersionResolver, VersionConfigResolver
Services/        Singleton services (IoCService, LogService, APIService, …)
Steps/           Reusable pipeline step wrappers (Phase, Hook, Event)
Validators/      ValidatorFactory
Enums.ts         All enums (HandlerTypes, HookFunctionTypes, Relationships, …)
Interfaces.ts    All TypeScript interfaces
Types.ts         Type aliases (PhaseFunction, HandlerFunction, …)
constants.ts     DEFAULT_HANDLERS, DEFAULT_APP_CONFIG, step definitions, …
Model.ts         Base Model class
Server.ts        Server bootstrap class
```

---

## Core concepts

### Model

Every API resource is represented by a class that extends `Model`. Getters define its behaviour — never constructor assignments.

```typescript
import { Model, HandlerTypes } from "axe-api";

class User extends Model {
  get primaryKey() {
    return "id";
  }
  get table() {
    return "users";
  } // default: snake_case plural of class name

  get fillable() {
    return {
      POST: ["name", "email", "password"],
      PUT: ["name", "email"],
      PATCH: ["name"],
    };
  }

  get validations() {
    return {
      POST: { name: "required|min:1|max:100", email: "required|email" },
    };
  }

  get handlers() {
    return [
      HandlerTypes.PAGINATE,
      HandlerTypes.SHOW,
      HandlerTypes.INSERT,
      HandlerTypes.UPDATE,
    ];
  }
}

export default User;
```

Key `Model` getters:

| Getter        | Type                                                    | Default                            |
| ------------- | ------------------------------------------------------- | ---------------------------------- |
| `primaryKey`  | `string`                                                | `"id"`                             |
| `table`       | `string`                                                | plural snake_case of class name    |
| `fillable`    | `string[] \| IMethodBaseConfig<string[]>`               | `[]`                               |
| `validations` | `ModelValidation \| IMethodBaseConfig<ModelValidation>` | `{}`                               |
| `handlers`    | `HandlerTypes[]`                                        | all except FORCE_DELETE and SEARCH |
| `middlewares` | `ModelMiddleware`                                       | `[]`                               |
| `transaction` | `boolean \| IHandlerBasedTransactionConfig[]`           | `false`                            |
| `cache`       | `ICacheConfiguration \| IHandlerBasedCacheConfig[]`     | disabled                           |
| `serializers` | serializer array                                        | `[]`                               |

### HandlerTypes enum

```typescript
HandlerTypes.INSERT; // POST   /resource
HandlerTypes.PAGINATE; // GET    /resource
HandlerTypes.SHOW; // GET    /resource/:id
HandlerTypes.UPDATE; // PUT    /resource/:id
HandlerTypes.PATCH; // PATCH  /resource/:id
HandlerTypes.DELETE; // DELETE /resource/:id
HandlerTypes.FORCE_DELETE; // DELETE /resource/:id/force
HandlerTypes.ALL; // GET    /resource/all  (no pagination)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axe-api/axe-api](https://github.com/axe-api/axe-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
