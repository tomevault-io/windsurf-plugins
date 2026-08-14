---
trigger: always_on
description: <!-- BEGIN quarkus-agentic-scaffolding conventions (managed block; do not edit inside. Re-run /setup-agentic-scaffolding to update.) -->
---

<!-- BEGIN quarkus-agentic-scaffolding conventions (managed block; do not edit inside. Re-run /setup-agentic-scaffolding to update.) -->

# Quarkus + LangChain4j + AI Stack - Project Conventions
# Version: 0.19.1

These conventions apply whenever Codex or Bob writes, reviews, or configures code in a Quarkus +
LangChain4j project. They are always-on. Procedural scaffolding steps and starter code live in
the `scaffold-project` skill and its templates, not here.

---

## 1. Required tooling (mandatory)

These tools are prerequisites for this project, not suggestions. Do not work around their absence:
if a required tool is unavailable, stop and report it rather than falling back to model memory or a
generic web search.

- **Quarkus Agents MCP - required for every Quarkus task.** Project creation, extension selection,
  configuration, version checks, API usage, and troubleshooting MUST go through the Quarkus Agents
  MCP; never create a Quarkus project, add an extension, or answer a Quarkus question from model
  memory by hand. Before any Quarkus task, VERIFY the MCP is reachable - its `quarkus_*` tools are
  present and a cheap call (e.g. `quarkus_status`) succeeds. If the tools are absent or the call
  fails, STOP immediately: report exactly what is missing, point the user to
  `/setup-agentic-scaffolding` (and to restarting the session after registering it, since MCPs load
  at session start), and end the turn. A missing or unreachable MCP is never permission to proceed
  manually - do not fall back to the Quarkus CLI, Maven/Gradle archetypes, model memory, or web
  search, do not offer to "continue without it", and do not treat the stop as optional. The only
  exception is `/setup-agentic-scaffolding`, whose job is to install it.
- **context7 - required for external library and framework documentation.** Before relying on
  memory or web search for any library or framework API - LangChain4j included - you MUST look it
  up with `context7` first.
- **superpowers skills - use whenever applicable.** Invoke the relevant `superpowers` skill
  capabilities for the task at hand.

---

## 2. Java conventions

- **Java 25 is the minimum language level**, not a ceiling. Compile with
  `maven.compiler.release` set to at least 25 and adopt newer language levels freely. Document
  any project that must pin an older level and explain why (see section 6). One cap applies to
  native targets: GraalVM ships no releases for JDK 26, 27, or 28, so native-image stays on the
  JDK 25 baseline (with quarterly updates) until JDK 29 lands (September 2027) - projects that
  build a native binary keep `maven.compiler.release` at 25 until then
  ([GraalVM release-train announcement](https://medium.com/graalvm/accelerating-the-graalvm-release-train-26b0d7cff2ab)).
- **Default to Virtual Threads for I/O-bound and blocking concurrent work.** Platform threads
  are acceptable only when the runtime or a critical dependency forbids virtual threads (for
  example, a JDBC driver that pins the carrier). When a blocking AI or tool call must run inside
  a reactive endpoint, run it on a virtual thread rather than on the event loop.
- **Use Scoped Values in place of `ThreadLocal`** for request- or agent-scoped identity that
  must survive virtual-thread continuations, avoiding the leakage and inheritance pitfalls of
  `ThreadLocal`.
- **Structured concurrency for related subtasks.** For fan-out across related concurrent
  subtasks, prefer declarative parallelism (LangChain4j `@ParallelAgent` / `@ParallelMapperAgent`,
  see section 4) or explicit virtual-thread fan-out (`Thread.startVirtualThread(...)` or an
  `Executors.newVirtualThreadPerTaskExecutor()`), instead of ad-hoc executor coordination.
  `StructuredTaskScope` is the preferred structured-concurrency primitive where the project can
  enable it; note it is a Java preview feature (requires `--enable-preview`) with GraalVM
  native-image considerations, so adopt it only when the preview flag and the native target
  allow.
- **Prefer records, sealed types, and pattern matching where they clarify intent.** Use records
  for DTOs and value objects (they also minimize the GraalVM reflection surface), sealed
  interfaces for closed hierarchies such as event or result types, and pattern-matching `switch`
  over those hierarchies so the compiler enforces exhaustiveness.

---

## 3. Quarkus conventions

- **Import the platform BOMs; do not pin extension versions.** Import `quarkus-bom` and
  `quarkus-langchain4j-bom` at the same platform version and let the BOMs manage every extension
  and LangChain4j version.
- **CDI-first.** Use `quarkus-arc` and standard CDI (`@ApplicationScoped`, `@Inject`,
  `@Produces`) for wiring. Produce framework objects (retrieval augmentors, memory providers,
  embedding stores) from `@ApplicationScoped` producer beans.
- **REST and API surface.** Use `quarkus-rest` (Quarkus REST) with `quarkus-rest-jackson` for JSON
  (Jackson is the Quarkus default serializer), and expose `quarkus-smallrye-openapi` so endpoints
  are documented and explorable.
- **Errors leave the REST edge as RFC 9457 problem details.** Add `quarkus-http-problem`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eldermoraes/quarkus-agentic-scaffolding](https://github.com/eldermoraes/quarkus-agentic-scaffolding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
