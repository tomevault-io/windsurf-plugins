---
trigger: always_on
description: Entry point for anyone picking up this project. Read this first.
---

# AGENTS.md — proxy-monster

Entry point for anyone picking up this project. Read this first.

## What it is

proxy-monster is a self-hosted, open-source database access-control proxy for
MySQL and PostgreSQL. It enforces column-level access control — deterministic,
role-based masking and deny — and is lineage-aware: it follows sensitive values
through SQL expressions, functions, subqueries, and joins. Clients connect with
their normal tools (psql, mysql, JDBC) over the native wire protocol; the proxy
masks per role and denies anything it cannot prove safe (**fail-closed**).

The goal is a transparent, in-VPC, self-hostable access-control proxy you own
and can extend, friendly to CLIs and native clients.

## Engine priority

MySQL is the primary target and the correctness bar for shipping — build and
verify it first. PostgreSQL support is experimental and developed alongside;
today PostgreSQL also serves as proxy-monster's own control-plane store.
PostgreSQL-only edge cases — its transactional-DDL behaviors (in-transaction DDL
visibility, Sync/implicit-transaction rollback, deferred-constraint COMMIT
failure) — may be carried as documented known limitations rather than block a
MySQL milestone. Keep MySQL and PostgreSQL separate when splitting work or
weighing review findings, and weight MySQL accordingly.

## Layout

Modules:

- `goproxy/` — Go data-plane wire proxy: MySQL/PostgreSQL codecs, token auth,
  the per-statement `Decide` call, inline result masking, and the backend
  broker.
- `control-plane/` — Kotlin control plane: identity and roles, Cedar
  authorization, the catalog, the per-statement decision, and the admin +
  console API (HTTP and gRPC).
- `analyzer/` — the Go sqlglot-go lineage probe (`probe/`) and its JVM FFM
  binding (`jvm/`); emits each statement's required grants.
- `engine/` — shared Kotlin enforcement code the control plane calls: the
  JVM-side wrapper around the sqlglot-go analyzer, system classification (the
  dangerous-function and system-catalog manifests), SQL normalization, and the
  mask functions used when a stored approval result is viewed.
- `auth/` — Kotlin OIDC login and the MCP OAuth authorization server.
- `auditmon/` — Go audit-trail monitor: verifies the hash chain, anchors
  off-box, detects anomalies, and exports to a SIEM.
- `pmon/` — Go client daemon: connect with a saved password while it brokers a
  short-lived token upstream.
- `mysqlwire/` — Go MySQL wire-protocol codec library (shared by `goproxy` and
  `pmon`).
- `proto/` — protobuf contracts: the proxy↔control-plane gRPC surface and the
  analyzer FFM boundary.
- `web/` — the Next.js console (editor, policies, access, audit, admin).
- `deploy/` — sample seed SQL for the compose backends.
- `docs/` — per-workstream design docs.

Key docs:

- [`README.md`](./README.md) — project front door.
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — the components, topology, trust
  boundaries, and ports.
- [`DESIGN.md`](./DESIGN.md) — the design decisions and the
  decision-to-enforcement flow.
- [`INSTALL.md`](./INSTALL.md) — install, run locally, and deploy (local + AWS).
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — how to build, test, and contribute.
- [`SECURITY.md`](./SECURITY.md) — how to report a vulnerability.
- [`KNOWN_LIMITATIONS.md`](./KNOWN_LIMITATIONS.md) — accepted caveats and gaps.
- [`docs/README.md`](./docs/README.md) — the design-doc index, plus a summary of
  what's built.

## Stack

- The control-plane is Kotlin/JVM; the wire proxy and the lineage analyzer are
  Go. Kotlin/JVM was the original choice, for an in-process JVM lineage engine;
  after the engine moved to sqlglot-go the system is being ported to Go
  incrementally, and the control-plane is the last Kotlin component. The
  analyzer still runs inside the JVM through a Foreign Function & Memory binding
  to a Go c-shared library (JDK 24 — the Java compiler is pinned to
  `--release 24` and every module targets JVM 24).
- Lineage: the sqlglot-go probe in `analyzer/` resolves column lineage — it
  parses each statement and traces which source columns every output and
  predicate derives from. proxy-monster owns the probe; sqlglot-go is a library
  dependency.
- Enforcement: masking and deny. When a query touches masked columns the
  analyzer rewrites `SELECT *` to an explicit column list (a faithful,
  semantically-identical rewrite) so mask ordinals are fixed; the proxy then
  masks those columns inline on the result stream. Anything the analyzer cannot
  prove safe is denied. Masking is result-stream rewriting — the query is not
  rewritten to inject mask expressions.
- Identity: OIDC. The IdP's group claim provisions local group membership (via
  SCIM or JIT on first login); local groups map to roles through the
  `group_role` map. The IdP never mints roles directly. Okta is the reference
  provider; any OIDC IdP works ([docs/auth-model.md](./docs/auth-model.md)).
- Wire auth: SSO login plus a local broker daemon (`pmon`). Saved connections
  use a fixed localhost password while the daemon injects a short-lived token on
  the upstream hop. The daemon does not renew that token — a login lasts one
  token TTL, then `pmon login` again (see
  [DESIGN.md](./DESIGN.md#identity-and-broker)).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ridi-oss/proxy-monster](https://github.com/ridi-oss/proxy-monster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
