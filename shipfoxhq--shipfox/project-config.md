---
trigger: always_on
description: Read [CONTRIBUTING.md](CONTRIBUTING.md) before working on this project.
---

# Agent guidelines

Read [CONTRIBUTING.md](CONTRIBUTING.md) before working on this project.

## Running tasks locally

This project uses [mise](https://mise.jdx.dev/) to manage tool versions. `node`, `pnpm`, and `turbo` are all available in the shell: no `npx` needed.

```sh
# Install dependencies
pnpm install

# Build all packages
turbo build

# Check (lint + format + import sorting) / type-check / test
turbo check
turbo type
turbo type:emit
turbo test

# Scope to a specific package
turbo build --filter=@shipfox/api...

# Start local services (Docker required)
docker compose up -d

# Dev mode with hot-reload (apps only)
pnpm --filter=@shipfox/api dev
```

## Module exports and imports

Avoid broad barrel files inside modules. Prefer importing from the file that owns the
symbol, such as `#core/auth.js` or `#presentation/dto/user.js`, rather than
`#core/index.js` or another catch-all index.

Package root exports should stay intentionally small: export only shared entities and
functions that are part of the package's public API. Do not export internal DB helpers,
routes, auth wiring, or test-only utilities from a package root unless another package
is meant to depend on them directly.

## Codebase conventions

### Backend modules

Backend feature packages are composed as declarative modules. A feature package
typically exports a `ShipfoxModule` that declares its `database`, `routes`,
`auth`, `e2eRoutes`, `publishers`, `subscribers`, and/or `workers`; apps should
compose those module declarations rather than wiring feature internals directly.
Module initialization runs in array order, so list modules with shared database
dependencies before dependents.

API feature packages usually follow a layered shape:

```text
src/
  core/          Domain behavior, entities, providers, and typed errors
  db/            Drizzle schema, migrations, persistence functions, row mappers
  presentation/  Fastify routes, auth adapters, and DTO conversion
```

### HTTP routes

Define HTTP endpoints with `defineRoute`, Zod schemas, and named auth methods
from `@shipfox/node-fastify` / `@shipfox/api-auth-context`. Prefer route groups
for shared prefixes, plugins, and inherited auth instead of repeating those
concerns in each route.

### DTOs and API contracts

Public HTTP contracts live in sibling `*-dto` packages. Put Zod request/response
schemas, inferred DTO types, and public event names/payload types there so the
backend, client, and E2E helpers all share the same contract.

Use camelCase for internal domain objects and snake_case for external HTTP DTOs.
Keep the conversion centralized in `presentation/dto/*` files; route handlers
should call a mapper like `toProjectDto()` rather than manually shaping response
objects inline.

### Persistence and events

Drizzle schema files own row-to-domain mapping. A table file should define the
table, infer DB types, and export `toX()` mappers; higher layers should work with
domain objects rather than raw Drizzle rows where possible.

Outbox events are part of a module's public contract. Define event names and
payload maps in the module's `*-dto` package, write outbox events in the same
transaction as the state change, and register publisher tables on the module
declaration.

### Client packages

Client feature packages should expose both transport functions and React Query
hooks. Use `@shipfox/client-api` for JSON requests, auth refresh, and `ApiError`
handling; colocate query keys, raw request functions, and hooks in the feature's
`hooks/api/*` module.

### Form management

Client forms use `@tanstack/react-form` driven by the `*BodySchema` Zod schemas
from the matching `*-dto` package. Zod 3.24+ implements Standard Schema, so
schemas pass directly to TanStack Form's `validators`:

```ts
const form = useForm({
  defaultValues,
  onSubmit: async ({value}) => { /* mutation */ },
});

<form.Field
  name="email"
  validators={{onBlur: bodySchema.shape.email, onSubmit: bodySchema.shape.email}}
>
```

Do not add `@tanstack/zod-form-adapter` or write custom Zod adapters: the
adapter package is legacy (pinned to form-core v0.x) and unnecessary on v1+.

Render every labeled input through `FormField` from `@shipfox/react-ui`, using
`FormFieldInput` to inherit the field's id, `aria-invalid`, and
`aria-describedby` automatically:

```tsx
<FormField label="Email" id="email" error={fieldError(field)}>
  <FormFieldInput
    type="email"
    value={field.state.value}
    onChange={(event) => field.handleChange(event.target.value)}
    onBlur={field.handleBlur}
  />
</FormField>
```

Validation runs `onBlur` per field and `onSubmit` for the form. Show field
errors only after the field has been blurred or after a submit attempt. See
the `fieldError(field)` helper at the bottom of each page form for the
boilerplate.

Server errors are classified by a per-feature `errorToFormError(error)` pure
function in `form-errors.ts`. It returns either
`{kind: 'field', field, message}` (routed to
`form.setFieldMeta(field, prev => ({...prev, errorMap: {...prev.errorMap, onServer: message}}))`)
or `{kind: 'form', message}` (rendered in an `<Alert>` above the form).
Use the `onServer` slot in `errorMap` (not `errors` directly) because

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShipfoxHQ/shipfox](https://github.com/ShipfoxHQ/shipfox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
