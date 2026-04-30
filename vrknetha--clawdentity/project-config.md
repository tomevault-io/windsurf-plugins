---
trigger: always_on
description: Clawdentity is a mixed TypeScript + Rust monorepo that provides cryptographic identity and trusted relay infrastructure for agent-to-agent communication. The deployable surface is in `apps/` (registry, proxy, landing, generic agent skill), shared contracts/runtime libraries are in `packages/`, and the canonical operator/runtime implementation is in `crates/`. Design and delivery must assume operators may run Clawdentity in local or constrained/offline environments and integrate any agent runtime
---

# AGENTS.md

## 1) Project Overview
Clawdentity is a mixed TypeScript + Rust monorepo that provides cryptographic identity and trusted relay infrastructure for agent-to-agent communication. The deployable surface is in `apps/` (registry, proxy, landing, generic agent skill), shared contracts/runtime libraries are in `packages/`, and the canonical operator/runtime implementation is in `crates/`. Design and delivery must assume operators may run Clawdentity in local or constrained/offline environments and integrate any agent runtime through the stable local webhook relay contract.

## 2) Build Commands

### TypeScript (apps + packages)
Run from repository root:
- `pnpm install`
- `pnpm build`
- `pnpm test`

## DID Format Guidance
- Keep all DID construction/parsing inside `packages/protocol/src/did.ts` so the switch to `did:cdi` stays centralized and tests update automatically.
- Trust slices (AIT, CRL, registry ownership, proxy pairings, connectors) should call shared helpers (e.g., `parseDid`, `validateAgentDid`) instead of copying brittle string-prefix checks; this keeps role expectations tied to context when DID semantics depend on parsed entity and authority fields.
- When new DID authorities appear (registry-owned vs self-hosted), track their identifiers in configuration and rely on parsed `authority` metadata for routing/trust checks rather than scattering hardcoded strings.
- Keep group identifier parsing centralized in `packages/protocol/src/did.ts` via `parseGroupId` (`grp_<ULID>`); do not duplicate group-id parsing logic in services.

## Group Naming Guidance
- Use `group join token` as the canonical product/code/doc term.
- Do not use `group invite` naming for membership-scoped tokens.

## Execution Governance
- GitHub issues are the source of truth for sequencing, blockers, and rollout updates.
- Primary execution tracker: https://github.com/vrknetha/clawdentity/issues/74.
- Do not use local execution-order files as governance source.

Common quality checks:
- `pnpm lint`
- `pnpm -r typecheck`
- `pnpm check:file-size`

### Rust (crates workspace)
Run from `crates/`:
- `cargo check`
- `cargo clippy --all-targets`
- `cargo test`
- `cargo build`

## 3) Module Map

### Apps (deployable services)
- `apps/registry` - Cloudflare Worker HTTP API for humans, agents, invites, API keys, and revocation data.
- `apps/proxy` - Cloudflare Worker relay/proxy that verifies Clawdentity auth headers and enforces trust policy.
- `apps/agent-skill` - Generic runtime-agnostic agent adapter instructions.
- `apps/landing` - Public docs/site and generated `/agent-skill.md` + `/skill.md` adapter artifacts.

### Packages (shared libraries)
- `packages/protocol` - Canonical protocol models and signing/verification wire-contract definitions.
- `packages/common` - Shared utility layer (errors, helpers, validation glue, shared types).
- `packages/connector` - TypeScript connector client/runtime primitives for relay connectivity.
- `packages/sdk` - Developer SDK for identity operations, verification, auth flows, and integration helpers.

### Rust workspace crates
- `crates/clawdentity-core` - Core Rust library for identity, registry clients, connector/runtime, pairing, and persistence.
- `crates/clawdentity-cli` - Rust CLI binary and command surface for current operator workflows.

### Rust local test services
- `crates/tests/local/mock-registry` - Local mock registry used for integration and harness-style flows.
- `crates/tests/local/mock-proxy` - Local mock relay/proxy used for integration and connector testing.

## 4) CLI Commands

### Rust CLI (`crates/clawdentity-cli`)
- Help: `cargo run -p clawdentity-cli -- --help`
- Common ops: `cargo run -p clawdentity-cli -- init`, `register`, `whoami`, `agent create <name>`, `pair start <agent>`, `pair confirm <agent>`, `verify <token-or-file>`, `connector configure <agent> --delivery-webhook-url <url>`, `connector doctor <agent>`, `connector start <agent>`, `connector service install <agent>`

## 5) Deeper Docs
Use `docs/` as system of record:
- `docs/ARCHITECTURE.md` - end-to-end architecture across apps, packages, crates, and trust flows.
- `docs/MONOREPO.md` - workspace structure, dependency/build ordering, and cross-ecosystem testing strategy.
- `docs/DESIGN_DECISIONS.md` - architectural choices and tradeoffs.
- `docs/GOLDEN_PRINCIPLES.md` - non-negotiable quality constraints.
- `docs/HARNESS_ACTION_PLAN.md` - staged execution and quality-enforcement plan.

## 6) Quality Rules
- Follow `docs/GOLDEN_PRINCIPLES.md` for code and documentation changes.
- Keep modules small, testable, and dependency direction explicit.
- Favor actionable errors and stable machine-readable outputs.
- Run relevant TypeScript and Rust checks before commit (`pnpm build` and `cargo check` are baseline gates).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vrknetha/clawdentity](https://github.com/vrknetha/clawdentity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
