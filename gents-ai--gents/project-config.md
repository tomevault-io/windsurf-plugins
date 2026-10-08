---
trigger: always_on
description: Gents is a Rust agent runtime with DefraDB as its control plane. Configuration,
---

# Repository instructions

Gents is a Rust agent runtime with DefraDB as its control plane. Configuration,
requests, responses, sessions, tools, triggers, and schedules are documents.
The `gents` runtime crate is the core; the CLI, desktop app, protocol, and
schemas operate or expose its vocabulary. Use DefraDB's DID identity and
document ACP for database authorization. Reuse its transaction and P2P APIs
through the existing Gents adapters.

## Configuration refactor stack

[PR #1430](https://github.com/gents-ai/gents/pull/1430) is the specification:
its description holds the decisions and deletion handoff; canonical config/protocol
structs define the target shapes. Old runtime code is evidence of existing behavior,
not permission to restore rejected abstractions.

- Keep the spec branch focused on types. It is intentionally red; per the user,
  do not run builds/tests or add compatibility code to make it compile here.
- Implement in stacked PRs: Lean [#1436](https://github.com/gents-ai/gents/pull/1436),
  then conformance [#1443](https://github.com/gents-ai/gents/pull/1443), then
  Rust/schema/loading and packs/consumers. Each PR targets its immediate parent.
  Keep its description current with baseline, owners, deletions and validation.
- Trace removed concepts through writers, readers and contracts. Preserve their
  guarantees through existing owners; delete obsolete implementations in their
  owning layer. Do not duplicate configuration types or graph selection paths.
- Remove historical migration steps, frozen old schemas and transition-specific
  tests; retain useful migration infrastructure. No legacy conversions or data wipes.
- Resolve spec conflicts in #1430 before propagating them. Validate each descendant
  and the integrated stack; the non-buildable foundation must not merge alone.

## Canonical output stack (#1571)

`gents_protocol::output` and the SDL are the breaking specification; the
[ownership table](contracts/canonical-output.md) assigns the remaining deletions.
Keep this layer intentionally red: no builds/tests or compatibility shims.
Stack Lean, generated conformance, then implementation/consumers, each on its
parent. Model the new contracts before implementing transitions or projections.
The foundation must not merge alone; validate the integrated stack before pushing
its implementation layers.

## Session-message stack (#1938)

The session-message spec layer (`gents_protocol::request_admission`, the
AgentRequest/AgentToolCall/AgentPrincipal SDL, `SubagentTools` and
`Proofs/Request/CausalHop.lean`) removes the privileged subagent position.
Keep it intentionally red: no Rust builds, tests or compatibility shims. Stack
L1 spec, L2 conformance, L3 runtime, L4 CLI/shims, L5 desktop, each on its
parent. The spec layer never merges alone; validate the integrated stack
before pushing its implementation layers.

## Foundation

For changes to legal transitions, invariants, or provider input, work in this
order:

1. Update the Lean model in `crates/gents/proofs/` and keep it free of `sorry`.
2. Update the generated or model-driven conformance tests.
3. Make the Rust implementation satisfy the contract.

Plumbing and tooling need no proof change when they preserve semantics. The
[proof map](crates/gents/proofs/README.md) identifies the modeled surfaces.

Conformance has one semantic source: the executable Lean owners. Serialize actual
modeled inputs/actions and derive expectations through those owners; do not keep
separate fixture constants or test-only Rust policy machines that mirror their behavior.
Explicit regression expectations must be checked against model execution. Wire
decoders and native adapters translate representations, not redefine policy.

## Ownership

- Core is bounded to the owned agent loop and task and data management in
  DefraDB: documents, triggers, tasks, workflows, identity and ACP. Other
  capabilities (OCR, git and GitHub, web research, domain tools) ship as WASM
  plugins in packs (gents-ai/packs) and run as tools or workflow stages; core
  adds only the plugin seam a pack needs, not the capability.
- Request state is only `lifecycle_state`, using
  `gents_protocol::request_lifecycle::RequestLifecycleState`. Claimed work runs
  through the owned completion loop; reuse its lifecycle and terminal owners.
- The owned loop is the sole provider-input boundary. Durable transcripts may
  be permissive; sanitize and narrow them there.
- DefraDB authenticates actors as DIDs and enforces document authorization
  through ACP. Keep Gents principals bound to those DIDs; do not add a parallel
  identity or authorization layer. Behaviors are reusable interfaces. The
  operating convention is one active runtime per principal; enforcement is
  deferred. Do not add host identity, leases, or host-migration machinery. Validate
  configured paths/resources through existing owners and fail affected work
  clearly when unavailable. Paths/observations are not portable identity data.
- Tools, MCP services, subagent targets, skills, tasks, callbacks, schedules,
  and event sources are documents. Tool groups are nested settings within Tools.
  Extend existing reconcilers and owners instead of adding side channels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gents-ai/gents](https://github.com/gents-ai/gents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
