---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`plinth` (`yi.shi:plinth`, version `jre-21`) is a lightweight, single-binary web framework built on **Guice 7 + Jetty 12 (ee10)** — deliberately **not** Spring Boot. It is intended for quickly standing up lightweight monolithic web apps. It is not published to Maven Central; consumers `mvn install` it locally. The repo doubles as the framework source and a runnable demo (`yi.shi.plinth.App` + `demo.HelloWord`).

- **Java 21** is required (`maven.compiler.source/target = 21`).
- The framework itself is request-dispatch + DI + Jetty bootstrapping. DB access is built in via `DataSourceModule` (MyBatis-Guice); register it with `ServiceBooter.startFrom(...)` to enable `@Mapper` scanning.

## Common commands

```bash
mvn clean install          # build + install to local Maven repo (required for consumers)
mvn compile                # compile only
mvn test                   # run JUnit Jupiter tests
mvn test -Dtest=AppTest    # run a single test class
mvn exec:java -Dexec.mainClass=yi.shi.plinth.App   # run the demo app (or run App.main() in an IDE)
```

Notes:
- Tests: `AppTest` is a placeholder; `DataSourceModuleTest` and `UserModuleTest` give real H2-backed coverage of the DB and user modules.
- The fat-jar / shade / assembly `<build>` plugins shown in `README.md` are **not** present in `pom.xml`. `mvn package` produces a plain jar; copy the build plugin block from the README if you need a runnable fat jar.
- Config is loaded into `System.getProperties()`, so JVM `-D` flags override file values at runtime.

## Architecture

### Startup flow
`App.main()` → `ServiceBooter.startFrom(mainClass, modules...)`:
1. Static block registers `JettyModule`; any user `Module`s passed to `startFrom` are also registered with `ModuleRegister`.
2. `@PropertiesFile` on the main class names config files (classpath paths **or** `http(s):` URLs) loaded by `CoreProperties` into `System.getProperties()`.
3. `IocModule.registScanPackage(mainClass)` scans the main class's package for `@HttpService` classes.
4. `ModuleRegister.getInjector()` builds the Guice injector (`Stage.DEVELOPMENT`) from all registered modules, resolves a `JettyBootService`, and starts the Jetty `Server`.

`JettyModule` wires a `ServletContextHandler` at `/` mapped to `DispatcherServlet` + Guice's `GuiceFilter`, plus a separate static-file `ResourceHandler` context. `GuiceServletCustomContextListener` is registered as a servlet listener.

### Request dispatch flow
`DispatcherServlet.service()` → `ServletHelper.init()` (stores req/resp in a `ThreadLocal`) → initializes SA-Token context/DAO → dispatches by HTTP verb to `RestApiServiceImpl.doGet/doPost/...`.

`RestApiServiceImpl` is constructed once (via `DispatcherServlet.initRestApiService()`, triggered from `IocModule`'s constructor). At construction it reflects over all `@HttpService` classes and builds **per-HTTP-verb maps** keyed by `@HttpPath` value, plus parameter (`@HttpParam`) and request-body (`@HttpBody`) metadata. Duplicate paths throw `ReduplicativeMathodPathException`.

Per request, `invoke()`:
1. Matches `requestURI − contextPath` against the verb's path map (404 if no match).
2. Runs `@AUTH` checks via SA-Token (login + `andRole`/`orRole`).
3. **Instantiates a fresh controller instance by reflection** (`ReflectionUtils.newInstance` — requires a no-arg constructor) and **manually field-injects** `@Inject` (Guice or `jakarta.inject.Inject`) and `@Properties` fields. Controllers are **per-request, not Guice-managed singletons** — no constructor injection or AOP applies to them.
4. Binds `@HttpParam` (query params) / `@HttpBody` (JSON body deserialized via Jackson) and invokes the method.
5. `HttpRespHelper` serializes the return: a `ReturnType` (`JSON<T>`, `HTML`, `BINARY`) is honored; any other object is auto-wrapped in `JSON` and serialized to JSON.

### Two Guice injectors (important)
There are **two independent injectors** — do not assume one:
- `ModuleRegister.getInjector()` — built from `JettyModule` + user modules. Used for Jetty boot and for **controller field injection** inside `RestApiServiceImpl`.
- `GuiceServletCustomContextListener.getInjector()` — built from `ServletModule` (which installs `IocModule`); used by `GuiceFilter` for request/servlet scoping. Constructing `IocModule` (a static field of `ServletModule`) is what triggers the controller scan and `DispatcherServlet.initRestApiService()`.

### Defining an endpoint
- `@HttpService` on the class.
- `@GET`/`@POST`/`@PUT`/`@DELETE`/`@HEAD`/`@OPTIONS` + `@HttpPath("/path")` on the method. (Multiple method annotations can stack on one method.)
- Parameters: `@HttpParam("name")` for query params, `@HttpBody` for a single JSON request body (only one `@HttpBody` per method).
- `@AUTH(orRole=, andRole=, authUrl=)` for SA-Token-gated access; `authUrl` redirects on auth failure, otherwise 401.

### Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiyi25928988/minio-shell](https://github.com/shiyi25928988/minio-shell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
