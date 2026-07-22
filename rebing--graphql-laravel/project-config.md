---
trigger: always_on
description: A Laravel wrapper for `webonyx/graphql-php`. PHP 8.2+, Laravel 12+.
---

# Agent Guidelines for graphql-laravel

A Laravel wrapper for `webonyx/graphql-php`. PHP 8.2+, Laravel 12+.

For development setup, commands, code style, and the PR workflow, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Project Structure

```
src/                          # Production code (Rebing\GraphQL namespace)
├── GraphQL.php               # Core class: schema building, query execution, type registry
├── GraphQLController.php     # HTTP controller for GraphQL requests
├── GraphQLServiceProvider.php # Service provider (config, routes, commands)
├── Console/                  # 12 artisan make:graphql:* generators + stubs
├── Error/                    # GraphQL-layer errors (client-safe, extend graphql-php Error)
├── Exception/                # PHP exceptions (config errors, not client-safe)
└── Support/                  # Base classes, middleware, contracts, tracing, pagination
    ├── Type.php / Field.php / Query.php / Mutation.php  # Core hierarchy
    ├── InputType.php / EnumType.php / InterfaceType.php / UnionType.php
    ├── ExecutionMiddleware/  # Pipeline middleware for full execution
    ├── Tracing/              # OpenTelemetry observability
    ├── Contracts/            # TypeConvertible, ConfigConvertible
    └── Facades/              # GraphQL facade

bin/                          # Dev scripts (standalone PHP, no autoload)
└── update-bc-baseline.php    # Generates/replaces baseline XML from BC checker output

tools/                        # Isolated dev tool installations (separate composer roots)
└── bc-check/                 # roave/backward-compatibility-check (own composer.json, no lock committed)

tests/                        # Two PHPUnit suites
├── TestCase.php              # Base: extends Orchestra Testbench, no DB
├── TestCaseDatabase.php      # Base: adds SQLite in-memory DB + migrations
├── Unit/                     # Unit tests (no database)
├── Database/                 # Database tests (SQLite in-memory)
└── Support/                  # Shared fixtures: Models/, Objects/, Queries/, Types/, Traits/

config/config.php             # Publishable Laravel config (schemas, types, middleware, security, tracing)
```

## Architecture

### Class Hierarchy

```
Field (abstract)           # Core: authorize(), rules(), args(), type(), resolve()
├── Query                  # Semantic alias (empty, extends Field)
└── Mutation               # Semantic alias (empty, extends Field)

Type (abstract)            # Base for all GraphQL types: fields(), attributes(), toType()
├── InputType              # → InputObjectType
├── EnumType               # → EnumType
├── InterfaceType          # → InterfaceType (adds resolveType/types)
├── UnionType              # → UnionType (abstract types())
└── UploadType             # → ScalarType (file uploads)
```

### Two Middleware Layers

**Execution Middleware** — wraps the full GraphQL execution pipeline:
- Base: `AbstractExecutionMiddleware` in `src/Support/ExecutionMiddleware/`
- Signature: `handle(string $schemaName, Schema $schema, OperationParams $params, $rootValue, $contextValue, Closure $next): ExecutionResult`
- Built-in: `ValidateOperationParamsMiddleware`, `AutomaticPersistedQueriesMiddleware`, `AddAuthUserContextValueMiddleware`, `UnusedVariablesMiddleware`, `GraphqlExecutionMiddleware` (terminal, always last)
- Configured via `graphql.execution_middleware` config or per-schema

**Resolver Middleware** — wraps individual field resolvers:
- Base: `Middleware` in `src/Support/Middleware.php`
- Signature: `handle($root, array $args, $context, ResolveInfo $info, Closure $next)`
- Per-field: `$middleware` property on `Field`/`Query`/`Mutation`
- Global: `graphql.resolver_middleware_append` config

### Key Classes

- **`GraphQL`** (`src/GraphQL.php`): Singleton, uses `Macroable`. Type registry with modifier parsing (`GraphQL::type('[User!]!')`). Schema building, query execution, error formatting.
- **`Privacy`** (`src/Support/Privacy.php`): Abstract base for field-level access control.
- **`ConfigConvertible`** (`src/Support/Contracts/ConfigConvertible.php`): Interface for class-based schema definitions (returns array with `query`, `mutation`, `types`, `middleware`, etc.).
- **`ResolverParameterInjector`** (`src/Support/Contracts/ResolverParameterInjector.php`): Interface for extensible resolver DI. External packages register injectors via `Field::registerParameterInjector()`.

### Resolver Conventions

- `Field::resolve()` supports dependency injection: `ResolveInfo`, or any container-resolvable class. External packages (e.g. `rebing/graphql-laravel-select-fields`) can register custom injectors via `Field::registerParameterInjector()`.
- Types auto-discover resolvers via `resolve{StudlyFieldName}Field()` methods on the Type class
- Authorization: override `authorize()` on `Field`/`Query`/`Mutation`
- Validation: override `rules()` to return Laravel validation rules for args

## Test Conventions

### Base Classes

- **`TestCase`** (`tests/TestCase.php`): Extends Orchestra Testbench. Provides `$this->queries` and `$this->data` fixtures, `httpGraphql()` helper, schema assertion methods. Pre-configures `default`, `custom`, and `class_based` schemas.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rebing/graphql-laravel](https://github.com/rebing/graphql-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
