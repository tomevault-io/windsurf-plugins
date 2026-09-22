---
trigger: always_on
description: This file defines repository-wide rules for AI coding agents and human contributors making architectural changes to SQLBraid.
---

# AGENTS.md

This file defines repository-wide rules for AI coding agents and human contributors making architectural changes to SQLBraid.

Architecture orientation is captured in `docs/mental-model.md` (with the Korean
translation in `docs/mental-model.ko.md`). Durable authority is split by boundary:
application compatibility is recorded in `docs/public-api-audit.md`, driver
implementation rules in `docs/driver-author-guide.md`, release mechanics in
`docs/SQLBraid_release_readiness.md` and the release workflows, and user-facing
behavior in the README/website. Historical planning artifacts are context, not
authority.

---

## 1. Product identity

SQLBraid is a **SQL-first data-access toolkit for TypeScript**.

Canonical authoring:

```ts
const query = sql.rows<UserRow>`
  SELECT id, name
  FROM users
  /*@braid where*/
    /*@braid if ${name != null}*/
      AND name = ${name}
    /*@braid end*/
  /*@braid end*/
`;
```

The project is not an ORM, not query-builder-first, not a validator framework and not a parity implementation of another database library.

---

## 2. Architecture priority

Prefer, in order:

1. SQL-first authoring;
2. safe bound parameters;
3. readable/local dynamic SQL;
4. explicit application contracts;
5. Standard Schema for one-row result mapping;
6. thin dialect/driver boundaries;
7. explicit physical-connection ownership;
8. observable execution rather than hidden middleware magic;
9. offline ordinary development;
10. optional metadata/codegen outside runtime/compiler;
11. deletion/simplification over partial SQL semantics.

---

## 3. Do not rebuild database semantics

PV3 removed the broad SQL AST/resolver. Keep it removed.

Do not add complete SQL grammars, function catalogs, operator/coercion systems or arbitrary SQL-to-TypeScript inference.

When a requirement can be solved by explicit contracts, Standard Schema, driver metadata, codegen or a narrow lexical check, use that smaller mechanism.

---

## 4. Result contracts and mapping

### 4.1 Explicit contract

```ts
sql.rows<UserRow>`SELECT ...`;
```

The developer owns SQL/result correspondence unless runtime validation/mapping is attached.

### 4.2 Query-bound result mapping

```ts
sql.rows(UserSchema)`SELECT ...`;
```

uses the official Standard Schema protocol. The schema output type is the row type.

Pipeline:

```text
driver row
 -> dialect TypePolicy normalization
 -> query-bound Standard Schema
 -> optional execution-level schema
 -> application row
```

Use `@standard-schema/spec`. Do not maintain a private protocol clone and do not make Valibot, Zod, ArkType or another implementation a runtime dependency.

### 4.3 One row to one value

Result mapping may validate, transform, parse JSON/text and create temporal/domain values. It must not introduce ORM graph assembly, identity maps, relation hydration or entity lifecycle.

### 4.4 Input mapping remains outside the runtime contract

Do not implement an application input-codec framework unless the user explicitly changes the roadmap.

Ordinary `${value}` remains a driver-bound value.

---

## 5. Dynamic SQL/compiler rules

Supported directive namespace: `/*@braid ...*/`.

Supported v1 directives: `if`, `choose`, `when`, `otherwise`, `where`, `set`, `trim`.

Guarded lowering must preserve:

- lexical `this`;
- evaluation order;
- once-only evaluation;
- inactive-branch laziness;
- TypeScript narrowing;
- source maps/directive prologues;
- query-bound mapper identity/output typing.

Do not add exponential variant proof.

---

## 6. Bind and structural SQL

Ordinary interpolation is always bound.

Structural SQL requires explicit APIs:

```ts
sql.ident(...)
sql.fragment`...`
sql.list(...)
sql.join(...)
sql.raw(...)
```

`sql.raw()` is trusted/unsafe. Security regressions here are release blockers.

### 6.1 Logical statement and binding boundary

Template/core rendering produces an immutable `RenderedStatement`:

```ts
interface RenderedStatement {
  readonly segments: readonly string[];
  readonly parameters: readonly RenderedParameter[];
  readonly resultKind: QueryResultKind;
  readonly dialectId: string;
  readonly fingerprint?: string;
  readonly variantFingerprint?: string;
}
```

`segments.length === parameters.length + 1`. Each `RenderedParameter` is one
value plus optional interpolation and `ParameterTypeHint`. Structural helpers
are merged into segments. Keep this value-only boundary intact: a parameter is
never raw SQL, an identifier, nested query, driver fragment or tagged-template
command. `RenderedStatement` is the execution source of truth; do not retain
parallel mutable text/values/hints/maps.

Dialects describe SQL lexical/quoting/type behavior. They do not generate
placeholders. Driver packages implement `StatementBindingAdapter.describe()`
and own `text-positional`, `text-named`, `typed-request` or
`native-value-template` materialization. Use `parameterizedSql(statement,
placeholder)` only as a derived view. Read
[`docs/driver-author-guide.md`](docs/driver-author-guide.md) before adding a
custom executor, provider or binding adapter.

Binding description is pure and pre-acquire. `QueryExecutor` and
`ConnectionProvider` expose the same `statementBinding` object; leases must

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clickin/SQLBraid](https://github.com/Clickin/SQLBraid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
