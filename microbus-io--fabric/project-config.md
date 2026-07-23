---
trigger: always_on
description: This repository is the **Microbus framework** itself - the foundation that downstream application projects import and build on. You are working on framework internals, not on an application built with the framework.
---

This repository is the **Microbus framework** itself - the foundation that downstream application projects import and build on. You are working on framework internals, not on an application built with the framework.

## Two kinds of code in this repo

1. **Framework packages** - `connector/`, `service/`, `application/`, `transport/`, `frame/`, `httpx/`, `pub/`, `sub/`, `cfg/`, `env/`, `lru/`, `dlru/`, `mem/`, `openapi/`, `trc/`, `utils/`, `setup/`, `workflow/`. These are library code consumed by every Microbus application. Changes here affect all downstream projects.

2. **Microservices built with the framework** - `coreservices/` and `exampleservices/` contain microservices that follow the same conventions as any downstream application. When working in these directories, follow the patterns and skills described in `.claude/rules/microbus.md`.

## Working on framework packages

- **Public API surface matters.** Exported types, functions, and interfaces in framework packages are consumed by downstream projects. Avoid breaking changes to exported signatures.
- **`connector/`** is the backbone - it implements the messaging bus, subscriptions, lifecycle, and configuration machinery that every microservice relies on.
- **`service/`** builds on `connector/` to provide the higher-level `Service` base type with convenience methods (`LogInfo`, `DistribCache`, `Now`, etc.).
- **`application/`** handles microservice orchestration, startup/shutdown sequencing, and the test harness (`RunInTest`).
- **Tests** for framework packages are standard Go unit tests (`go test ./connector/...`), not the microservice integration test pattern used in `coreservices/` and `exampleservices/`.

## Working on coreservices/ and exampleservices/

These directories contain microservices built using the framework. Treat them the same way you would treat microservices in a downstream application project:

- Follow all conventions in `.claude/rules/microbus.md`
- Use the skills in `.claude/skills/` for scaffolding and adding features
- Respect `MARKER` and `HINT` comments
- **`myserviceapi/definition.go` is the source of truth; `manifest.yaml` and the boilerplate are derived from it.** `cmd/genservice` regenerates `manifest.yaml`, `myserviceapi/client.go`, `intermediate.go`, `mock.go`, and `mock_test.go` from `definition.go` (the housekeeping skill runs it); never hand-edit those generated files, hand edits are overwritten. The manifest exists as a fast navigational map for coding agents - what a microservice exposes - so an agent can model the system without reading every file. After changing `definition.go`, regenerate rather than editing the derived files.
- **The generated `ToDo` interface in `intermediate.go` must stay an interface.** `NewIntermediate(impl ToDo)` takes an interface, not a concrete `*Service`, so one constructor serves both `*Service` in production and `*Mock` in tests; it is also the compile-time proof that both implement every handler. genservice derives it from the feature set, so there is nothing to hand-edit - but do not redesign the generator to take a concrete type, which would break mocking.

## The token minters are trust roots

The `Mint` endpoints on `access.token.core` and `bearer.token.core` sit on port `:666` and sign whatever claims the
caller supplies (only `iss`, `idp`, `iat`, `exp`, `jti` are stamped by the service). This is deliberate: it is what
makes claims enrichment and actor impersonation work. It also means any caller that can publish to `:666/mint` can
mint any claims, including administrative roles.

Do not try to secure a `Mint` endpoint by adding `requiredClaims` or an in-handler check that the caller already
holds some claim. That is circular: obtaining any verified claim requires a token, and issuing a token is exactly
what `Mint` does. A trust root cannot be gated by the credential system it roots. The only technical control is the
NATS `:666` `PUB` ACL, which `cmd/gencreds` grants a microservice solely because its source code calls a `:666`
endpoint; the operational controls (isolating trust-root microservices, and a CI allow-list that fails the build on
an unlisted `:666` grant) live in the production deployment guide's "Hardening the Trust-Root Tier" section, not in
framework code. The ingress token exchange is not a safer mint either: it verifies and issuer-pins the bearer token
and then passes its claims into `Mint` verbatim, so the access token is exactly as trustworthy as the bearer token,
and what each token carries is application policy rather than a framework-enforced invariant.

When reviewing or changing framework code, treat any new `:666` call site as a trust-root capability expansion, and
never rework a minter to gate itself on a claim.

## Authoring framework upgrade skills

Every fabric release ships exactly one upgrade skill, [upgrade-microbus](.claude/skills/microbus/upgrade-microbus),
and it is self-propagating: its Phase 1 migrates a project by the single increment `SOURCE -> DEST`, and its Phase 2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microbus-io/fabric](https://github.com/microbus-io/fabric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
