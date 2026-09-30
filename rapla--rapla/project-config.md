---
trigger: always_on
description: **Rapla** is a Java-based resource scheduling and event planning application (v3.0-SNAPSHOT, AGPL/Apache2). It uses Maven, targets Java 17, runs on Java 21. Key technologies: Spring Boot 4.0 (Tomcat 11), Jackson 3, Swing, JAX-RS / Spring MVC, RxJava3, iCal4j 4.2, Exchange Web Services.
---

# AGENTS.md - Opencode Rules

## Project Overview

**Rapla** is a Java-based resource scheduling and event planning application (v3.0-SNAPSHOT, AGPL/Apache2). It uses Maven, targets Java 17, runs on Java 21. Key technologies: Spring Boot 4.0 (Tomcat 11), Jackson 3, Swing, JAX-RS / Spring MVC, RxJava3, iCal4j 4.2, Exchange Web Services.

**Jackson 3** (PRD 011, done): the runtime is Jackson 3 — databind/core live in the `tools.jackson.*` package, **not** `com.fasterxml.jackson.*`. Annotations (`@JsonIgnore`, `@JsonProperty`, …) stay in `com.fasterxml.jackson.annotation.*` (the version-shared package) — those imports are correct. Only `tools.jackson.databind.ObjectMapper` is the rapla mapper. Rapla uses only Jackson 3; every use of Jackson 2 (`com.fasterxml.jackson.databind`/`.core`) is forbidden except the build-time OpenAPI spec generation (springdoc, test scope only — never in the runtime classpath).

**Deployment topology:** the server is multi-pod capable — multiple instances run against one shared store, coordinating via the update history (JSON change records, polled ~every 10 s) plus store-level locking; don't assume a single instance or a process-local shared cache (§8's "one server per checkout" is a dev convention only). Lock layers (process / resource / global) and the multi-pod concurrency model: [`docs/architecture/locking.md`](docs/architecture/locking.md).

The codebase is a **5-module Maven reactor** (PRD 005, 2026-05-07) plus a **separate Angular SPA tree** (PRD 026, 2026-05-12):

| Module / tree | Role |
|---|---|
| `rapla-bom` | BOM + parent POM (versions, plugin config). Was `parent/`. |
| `rapla-core` | Shared layer: entities, facade, framework, scheduler, storage interfaces, REST DTOs/endpoint interfaces, components/{util,layout,restproxy,i18n}, logger, inject. NO Spring Boot, NO Swing. |
| `rapla-client` | Swing client + presenters: `org.rapla.client.*`, components/{calendar,calendarview,iolayer,tablesorter,treetable}, plugin `*/client/*` and `*/swing/*`. Uses `spring-context` only — explicit `AnnotationConfigApplicationContext`, NOT `@SpringBootApplication`. Depends on rapla-core. |
| `rapla-server` | Server: `org.rapla.server.*` (excl. the `RaplaSpringBootApplication` entry point), JDBC storage, REST controllers, Spring autoconfig (`META-INF/spring/AutoConfiguration.imports`), plugin `*/server/*`. Depends on rapla-core only (verified 2026-06-10 — the PRD 005 D3 rapla-client compromise is resolved; `RaplaBuilder`/abstractcalendar live in rapla-core). |
| `rapla-app` | Runnable Spring Boot application: `RaplaSpringBootApplication`, `application.yml`, `src/assembly/`, `src/main/distribution/`, signing profiles, JNLP webclient/ staging. Produces `rapla.jar` (Spring Boot fat JAR). |
| `rapla-angular/` | Angular SPA — separate tree, **not in the Maven reactor**. Built with `npm`, served at `/app/` in dev (via `ng serve` proxy on :4200) and prod (via Spring Boot static handler). Talks to the rapla-app REST API at `/api/*`. Has its own `package.json`, Vitest tests; talks to the server via `HttpClient` (auth + `/api/graphql`), no generated client. See AGENTS.md §14 + the `angular-frontend` skill. |

The repo-root `pom.xml` is the reactor aggregator (artifactId `rapla-aggregator`, packaging=pom, lists the five Maven modules); running `mvn` from the repo root walks the whole reactor.
`rapla-angular/` lives outside the reactor entirely — it's an Angular project, not a Maven module; build/test commands are `npm`, not `mvn`.

**Build & Test:**
- Reactor compile: `mvn compile`
- Reactor test: `mvn test`
- Per-module compile: `mvn -pl rapla-server -am compile`  *(use `-am` to also build module deps in-tree; otherwise Maven looks in `~/.m2/repository`)*
- Per-module test: `mvn -pl rapla-server -am test`
- Targeted test: `mvn -pl rapla-app -am test -Dtest=RaplaSpringBootApplicationTest`
- Run the dev server: `mvn -pl rapla-app -am spring-boot:run -Dspring-boot.run.fork=false -Dspring-boot.run.profiles=local` *(must use `-am` from repo root — see §5 hard rules; do not `cd rapla-app`, do not `mvn install`)*
- Full server lifecycle (background, PID/logs, graceful stop): see §8 below
- Test the deployable fat JAR + signed JNLP webclient: load the **`test-deployment`** skill
- Probe the running server's REST API directly (login, getResources, queryAppointments, etc.) — load the **`api-testing`** skill
- Requires SDKMAN (Java 21 + Maven) on WSL2 Ubuntu

## Skills

Detailed how-tos live as **Agent Skills** under `.agents/skills/<name>/SKILL.md`, auto-discovered by `name` + `description` and loaded on demand — the rules below name the relevant skill where it applies (*"load the X skill"*). Claude Code sees them as `rapla:<name>` only when started with `claude --plugin-dir .`; other engines and the setup details: [`docs/development.md`](docs/development.md#agent-skills--how-engines-find-them). Restructuring this file or its skills: load the **`agents-cleanup`** skill.

## Rules

### 0. Session discipline — context budget + risky-change branching


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rapla/rapla](https://github.com/rapla/rapla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
