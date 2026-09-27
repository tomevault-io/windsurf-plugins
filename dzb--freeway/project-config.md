---
trigger: always_on
description: Freeway is a JDK 25+ multi-module Maven project. Keep changes scoped, explicit,
---

# Repository Guidelines

Freeway is a JDK 25+ multi-module Maven project. Keep changes scoped, explicit,
and convention over configuration. This file is the single home for repo-wide
conventions: build, module layout, naming, design rules, testing, commit rules.
Architecture boundaries and framework internals live in
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — read it before changing module
boundaries, hooks, config resolution or lifecycle.

## Build

Requires JDK 25+ (JUnit 6.1.3, SLF4J 2.0.18).

```
mvn test                            # all core modules
mvn -pl freeway-ioc test            # single module
mvn -pl freeway-http -am test       # module + its upstream deps
mvn -pl freeway-cloud -am test      # module + its upstream deps
mvn test -Dtest=CoercerDefaultTest  # one test class
```

Run `mvn -pl <module> -am test` (or `clean test`) after touching a module that
others consume: a stale installed snapshot hides ABI changes, and the failure
surfaces later as a `NoSuchMethodError` in a downstream module. Third-party
adapters (Undertow/Jetty engines, HikariCP, Kafka) live in
[freeway-ext](https://github.com/dzb/freeway-ext): `mvn install` the core first.

## Module Map

| Module | Purpose | Module dependencies |
|--------|---------|---------------------|
| `freeway-commons` | JSON, coercion, defer, scoped cache, validation, logging | — |
| `freeway-ioc` | Container, binding DSL, scopes, injection, extensions, symbol config | commons |
| `freeway-boot` | Launcher, runtime lifecycle, profiles, config cascade (hot reload) | ioc + commons |
| `freeway-http` | Routing, built-in HTTP engine, WebSocket, SSE | ioc + commons (+ boot, test) |
| `freeway-db` | JDBC, ORM, pooling, transactions, migrations | commons (+ ioc, DbModule only) |
| `freeway-flow` | Graph workflow engine — 7 node types, v3 DAG format (boot-validated), task vocabulary, branch isolation | ioc + commons |
| `freeway-cloud` | Cloud-native foundation — discovery, remote invocation (JDK HttpClient), observability, resilience, health, secrets, storage | ioc + commons + http (+ boot, test) |

No module adds an external dependency beyond SLF4J 2.0.18 (declared explicitly
by every module that logs — currently all but `ioc`, which inherits it
transitively) plus JUnit at test scope. Anything else belongs in an ext adapter.

## Naming

- Public interfaces use the domain name: `Container`, `JsonCodec`, `Route`.
- **`XDefault` vs `XImpl`** — the deciding question is whether the *outside can
  substitute* the implementation for that role. The operational test is one
  line: **can a module bind an alternative with `.primary()` and have the
  container honor it?** If yes the type is `XDefault`, no matter how concretely
  the framework itself wires it, and no matter that it lives in `internal`:
  - `XDefault` — it can: an extension binds an alternative with `.primary()`,
    an adapter builds on the default, or config activates another one.
    Examples: `AppRuntimeDefault`, `JsonCodecDefault`, `PoolDefault`,
    `FlowDriverDefault`, `FlowEngineDefault`, `ExchangeMetaDefault` and the
    twelve cloud defaults.
  - `XImpl` — it cannot: container-internal assembly (`ContainerImpl`,
    `BindingImpl`, `DatabaseImpl`), engine-internal components
    (`HttpContextImpl`), or per-owner types that coexist with other
    implementations (`PooledConnectionImpl`). A type stays `XDefault` even
    where the framework wires it concretely.
  - Substituting a role means honoring its whole seam, not just its lookups:
    contributions reach the config chain through one channel —
    `contribute(SymbolProvider.class)` into the extension store; there is no
    register/replay step. A replacement `SymbolSource` must take that view
    itself, in the factory that builds it:
    `binder.bind(SymbolSource.class).to(c -> new MySource(c.extension(SymbolProvider.class)))`.
    A replacement that ignores the view serves only its own tiers and boot's
    cascade disappears silently (`SymbolSourceReplacementTest` pins the
    pattern).
- **Factory verbs**: `of` builds a *value* from the parts you hand it — the
  records do this (`Endpoint.of`, `ServiceInstance.of`, `SymbolSpec.of`,
  `ModuleNode.of`), mirroring `List.of`. `create` is the framework *entry
  point* that hands you something to configure or run (`Freeway.create`,
  `FreewayApp.create`, `FlowEngine.create`, `Graph.create`), and `.builder()`
  is fluent assembly of a configured object — licensed *only* when the builder
  holds no defaults of its own (`FlowDriverDefault.Builder`: two required
  parts, nothing stated). An object that varies from a stated default in a
  field or two is a `defaults()` value plus per-field withers returning new
  instances — the wither verb is `withX` (`HttpServerConfig.withPort`,
  `CorsFilter.withAllowedOrigins`, `StaticResourceMount.withFallthrough`), while
  a bare noun form means a read (`StaticResourceMount.fallthrough()`), so a call
  site can always tell an assignment from a lookup — never a builder that copies
  those defaults into itself, which is a second owner of
  the same answer.
  Assembling a *service graph* is never a builder's job: `HttpServer.create(…)`
  is the one derivation of a server from its parts, `HttpModule` is its

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dzb/freeway](https://github.com/dzb/freeway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
