---
trigger: always_on
description: <!-- This file governs go/ and all of its subdirectories. -->
---

<!-- This file governs go/ and all of its subdirectories. -->
<!-- The repository-root AGENTS.md still applies. These Go-specific rules take precedence on conflict. -->

# Nyro Go

## Purpose and current state

`go/` is the Go implementation of Nyro AI Gateway. It includes the data plane,
standalone server, control plane, WebUI, configuration management, storage,
quota, telemetry, and administrative tooling.

The workload-neutral generation Host and the trusted LLM vertical slice are
implemented. Current requests use immutable configuration Snapshots, atomic
typed runtime generations, leases, explicit Protocol and Provider catalogs,
the fixed LLM pipeline, and generation-owned runtime resources.

Other module migrations are not complete. MCP, cross-domain Integration
composition, and Image, Audio, and Video workload runtimes are future designs
only. Do not claim their packages or capabilities exist. Preserve current
behavior while migrating other vertical slices incrementally.

## Architecture principle

> Varying capabilities are explicitly composed behind narrow typed contracts;
> resource owners have explicit lifecycles; workload-neutral invariants stay in
> the kernel; workload invariants stay in trusted runtimes.

This is static Go composition, not a dynamic plugin system. Do not use
`buildmode=plugin`, `.so` loading, or a third-party plugin ABI without a
separately approved design demonstrating a concrete requirement.

The implemented architecture is documented in
`docs/design/architecture.md`. Keep implementation and that document aligned.

## Kernel responsibilities

`internal/kernel` owns only workload-neutral invariants:

- component identity and dependency-graph validation;
- deterministic dependency ordering;
- lifecycle startup, rollback, retirement, and shutdown;
- typed Candidate and Host contracts plus atomic runtime-generation activation;
- leases that keep retiring generations alive until release; and
- readiness and runtime-generation status.

Kernel production code must use only the Go standard library. It must not
depend on:

- LLM or another workload type;
- request, response, stream, routing, retry, failover, or error semantics;
- protocols, providers, configuration parsing, or configuration transport;
- security, quota, telemetry, storage, Admin, WebUI, or HTTP transport; or
- global service locators, string-keyed dependency maps, or module discovery.

## Trusted LLM runtime

`internal/llm/runtime` owns all LLM-domain invariants: Canonical LLM IR
execution, mandatory phase order, routing authority, attempt isolation,
normalized errors, extension ownership, retry and failover, streaming commit,
health decisions, and trusted terminal delivery.

The phase order is fixed:

```text
Observe -> Resolve -> Authenticate -> Authorize -> Admit
        -> optional PreDispatch -> Dispatch -> optional PostResponse
        -> trusted terminal delivery -> reverse Finalizers
```

Optional phases may participate only at `PreDispatch` or `PostResponse`. They
must not change mandatory order, invoke the next phase, control terminal
delivery, bypass authorization or admission, or suppress reverse finalization.
Stream observers observe canonical deltas without controlling stream flow.

## Explicit composition and catalogs

Concrete Protocol Codecs and Provider Drivers are enumerated by
`internal/bootstrap`. Bootstrap constructs immutable typed catalogs, resolves
configuration, builds inactive Snapshot-bound runtime candidates and resources,
and submits their lifecycle graphs to the Kernel Host.

Importing a module package must not register, start, or activate it. Do not use:

- package `init()` for module registration or dependency wiring;
- blank imports whose purpose is Nyro module registration;
- mutable global registries populated as an import side effect;
- hidden discovery based on initialization order; or
- configuration I/O, goroutine startup, or resource acquisition from `init()`.

The reviewed blank import of `github.com/glebarez/go-sqlite` in
`internal/platform/database/sqlite` is a `database/sql` driver integration,
not module registration.

Tests must explicitly assemble the catalogs and dependencies they require.
Configuration may select only implementations present in an explicitly built
catalog; it must not cause otherwise unreferenced code to execute.

When adding a varying capability, define the smallest stable typed contract at
its consumer or in an established contract package. Do not introduce a service
locator, global container, general plugin framework, or speculative extension
point.

## LLM protocol and provider boundaries

The implemented request path is:

```text
Client Wire
    -> generic HTTP Server
    -> LLM HTTP Ingress
    -> Canonical LLM IR
    -> trusted Runtime Pipeline and Router
    -> Provider Driver extension
    -> Egress Codec
    -> Provider Driver preparation
    -> generation-owned Provider HTTP Transport
    -> Upstream
```

Responses and streams travel in reverse through their corresponding
boundaries.

### Ingress Codec

An Ingress Codec parses a northbound request, performs protocol-structural
validation, maps common semantics into the Canonical LLM IR, and encodes
canonical responses, errors, and stream deltas for its client protocol. It must

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyroway/nyro](https://github.com/nyroway/nyro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
