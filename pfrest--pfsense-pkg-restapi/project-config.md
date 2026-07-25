---
trigger: always_on
description: Contributor and AI-agent guide for **pfSense-pkg-RESTAPI** (https://pfrest.org).
---

# AGENTS.md

Contributor and AI-agent guide for **pfSense-pkg-RESTAPI** (https://pfrest.org).

This file is the **single source of truth** for how code in this repository is structured and how new contributions must be authored. It is intentionally opinionated. Read it end-to-end before proposing changes.

For deeper, file-scoped guidance see:

- `.github/skills/` — task-oriented playbooks (endpoints/models/fields, validators/commands/dispatchers, writing tests, full feature walkthroughs).
- `.github/instructions/` — GitHub Copilot path-scoped instructions that auto-apply when you edit specific directories (Models, Endpoints, Validators, Dispatchers, Tests).
- `docs/CONTRIBUTING.md` — human-facing contribution and build guide.
- `docs/BUILDING_CUSTOM_*` — long-form references for each subsystem.
- `https://pfrest.org/php-docs/` — generated PHPDoc for every class.

---

## 1. What this project is

`pfSense-pkg-RESTAPI` adds a fully featured REST and GraphQL API to **pfSense CE**. It is implemented as a FreeBSD package that installs PHP code under `/usr/local/pkg/RESTAPI` on a pfSense host.

The framework is **declarative and metadata-driven**: Endpoints, Models, Fields, Validators, Dispatchers, Auth, ContentHandlers, and Responses describe the system, and the runtime generates:

- HTTP endpoint PHP files in the pfSense webroot
- pfSense privileges (ACL entries) per endpoint and method
- OpenAPI 3 / Swagger documentation
- A GraphQL schema
- Background dispatcher cron jobs
- Optional webConfigurator forms

If you bypass the framework's metadata (e.g. by hand-writing handlers or hard-coding shapes), you silently break documentation, schema generation, privileges, HA sync, and tests. **Don't.**

---

## 2. Repository layout (only the parts you'll touch)

```
pfSense-pkg-RESTAPI/files/usr/local/pkg/RESTAPI/
├── Auth/              # Auth methods (BasicAuth, JWTAuth, KeyAuth)
├── Caches/            # Scheduled JSON dataset producers
├── ContentHandlers/   # Request/response Content-Type and Accept handlers
├── Core/              # Framework base classes (do not modify without maintainer approval)
│   ├── Auth.inc  Cache.inc  Command.inc  ContentHandler.inc
│   ├── Dispatcher.inc  Endpoint.inc  Field.inc  Form.inc
│   ├── Model.inc  ModelSet.inc  QueryFilter.inc
│   ├── Response.inc  Schema.inc  TestCase.inc  Tools.inc  Validator.inc
├── Dispatchers/       # Long-running / background workers
├── Endpoints/         # URL → Model adapters (thin)
├── Fields/            # Field types (StringField, IntegerField, ForeignModelField, ...)
├── Forms/             # webConfigurator form pages
├── Models/            # Feature logic (config schema + apply behavior)
├── ModelTraits/       # Cross-Model mixins (e.g. log file traits)
├── QueryFilters/      # Query operators (e.g. exact, regex, gt, lt)
├── Responses/         # Throwable HTTP responses (Success, ValidationError, ...)
├── Schemas/           # Schema generators (OpenAPISchema, GraphQLSchema, NativeSchema)
├── Tests/             # Custom *TestCase.inc files (see §10)
├── Validators/        # Reusable Field-level validation classes
├── autoloader.inc
└── .resources/
    ├── cache/         # Cache file output
    ├── schemas/       # Generated schema artifacts
    └── scripts/       # dispatch.sh, manage.php (build/runtime CLI)
```

Other top-level items:

- `tools/make_package.py` — builds the FreeBSD package.
- `vagrant-build.sh` + `Vagrantfile` — builds via a FreeBSD VM on your local machine.
- `docs/` — MkDocs source for https://pfrest.org.
- `composer.json` / `package.json` — runtime PHP deps and dev tooling (Prettier + plugin-php, Spectral).
- `.github/workflows/` — CI: Prettier, Black, phplint, phpdoc, package build, OpenAPI lint, runtests on real pfSense VMs.

---

## 3. Mental model: how a request flows

```
HTTP → generated PHP file in webroot
     → Endpoint (Auth → ACL → method dispatch → ContentHandler)
        → Model (validate Fields → run validators → validate_FIELD_NAME / validate_extra
                 → write to pfSense config (or call internal_callable) → apply())
           → apply() may spawn a Dispatcher process (filter reload, service restart, ...)
        ← Model returns representation
     ← Endpoint serializes via ContentHandler → Response
```

Two implications:

1. The Endpoint layer is essentially a router/adapter. Everything that _does_ something belongs in the Model (or in a Dispatcher invoked from the Model).
2. Field metadata is the schema. OpenAPI types, GraphQL types, validation, defaults, choices, sensitivity, pagination, and HATEOAS links all derive from Field/Model/Endpoint properties.

---

## 4. Non-negotiable rules

These are enforced by maintainers in code review.

1. **Model-first.** Every Endpoint sets `model_name` and exposes a Model. Endpoints **must not** override `get()`, `post()`, `patch()`, `put()`, `delete()` unless a maintainer has explicitly approved an exception.
2. **Field-driven schema.** Object shape, types, defaults, choices, constraints, sensitivity, conditional visibility, and help text live on `Field` objects in the Model constructor. Do not hand-write OpenAPI/GraphQL.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pfrest/pfSense-pkg-RESTAPI](https://github.com/pfrest/pfSense-pkg-RESTAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
