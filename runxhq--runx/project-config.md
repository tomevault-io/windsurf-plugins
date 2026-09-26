---
trigger: always_on
description: This file applies to the OSS workspace under `oss/`. It complements
---

# Runx OSS Conventions

## Scope

This file applies to the OSS workspace under `oss/`. It complements
`AGENTS.md`, `CLAUDE.md`, `docs/rust-kernel-architecture.md`, and
`docs/trusted-kernel-package-truth.md`.

## Contract Vocabulary

Governed runtime artifacts use the harness spine:

- `harness`: governed execution boundary with attenuated authority.
- `act`: contained payload with `intent`, `form`, and `closure`.
- `receipt`: sealed proof of a harness node.
- `decision`: accountable harness lifecycle choice.
- `signal`: world-before-action input.

Do not introduce compatibility aliases, `.v2` contract ids, or retired central
object names at governed boundaries. Product-facing skill names may remain
recognizable; wire contracts must use the spine vocabulary.

## Package Boundaries

Package names carry trust claims:

- contracts define portable schemas and generated validators.
- Rust `runx-core` owns pure state-machine and policy decisions.
- Deleted TypeScript core packages must not be restored as compatibility shims
  or build-only fallbacks.
- `runx-runtime` coordinates local execution, adapters, exact process
  invocation and supervision, caller interaction, and receipts.
- host adapters and protocol adapters touch external processes and protocols.
- `runx-cli` is the native command shell over the runtime.

OSS packages must not import cloud code. Core must not import runtime, adapter,
CLI, host-adapter, filesystem, network, or subprocess concerns.

## Connector and Tenant Neutrality

Portable Runx behavior targets provider capabilities, not connector vendors or
Runx-company tenancy. Skills and public contracts must not contain Nango
connection ids, provider-config keys, hosted tenant ids, connector endpoints,
or assumptions that credentials are held by Runx Cloud. The operator binds an
eligible local, self-hosted, third-party, or Runx-hosted connector at runtime.

Provider operations therefore have stable names, typed inputs, bounded result
projections, target-resource constraints, access classification, idempotency,
and readback semantics independent of the adapter that executes them. Hosted
adapters are optional implementations of those contracts. They must never
become prerequisites for an otherwise portable skill.

## Ownership Before Abstraction

Promote behavior into native code only for a runtime/security invariant or two
independent existing consumers. Keep domain policy in its owning skill, delete
the displaced path in the same cutover, and never expose fixture or package
layout through a generic API. `skills/skill-lab/SKILL.md` owns the full skill
authoring contract; `docs/skill-quality-standard.md` owns the review standard.

## Packet Artifact Eligibility

Every public runner must expose a complete inspectable input and output
contract. That does not make every runner or skill a global packet owner.

- Keep runner-specific nested values in `X.yaml` as ordinary JSON Schema.
- Put nested output shape in the producing output declaration's `schema`; the
  parser and runtime validate that same declaration and packet generation
  projects it without a second schema owner.
- Declare `packet: <packet-id>` only when the complete value crosses a named,
  reusable skill, runtime, SDK, provider, receipt, or registry boundary.
- Keep graph intermediates and one-run implementation values as typed step
  inputs or outputs; do not mint packet ids to make inspection work.
- A canonical Rust contract uses `public_packet_artifact` only when a native
  producer or public cross-language boundary requires distribution without an
  `X.yaml` producer. Existence in `runx-contracts` is not enough.
- One packet id has one schema owner. Consumers reference it and never copy its
  schema. Generated packet files are distribution outputs and must disappear
  when their last declaration or explicit public native owner disappears.
- A named packet schema describes its semantic fields or references another
  canonical contract. Bare `type: object`, unconstrained `{}`, and an opaque
  `additionalProperties` bag are not substitutes for a contract. An open JSON
  value is valid only as an explicitly named protocol extension or generic data
  payload inside an otherwise bounded semantic envelope.
- This rule is recursive. Every `type: object` in an input or distributed
  packet must declare structure or an explicit `additionalProperties` policy.
  Use `type: json` when the value is intentionally arbitrary JSON rather than
  disguising that intent as an object contract.

Review the active producer and consumer before adding or retaining a packet
artifact. An orphaned schema is legacy code, not compatibility evidence.

## Rust Bar

Rust code must keep the workspace green under:

```sh
cargo fmt --manifest-path crates/Cargo.toml --all --check
cargo clippy --manifest-path crates/Cargo.toml --workspace --all-targets -- -D warnings
cargo test --manifest-path crates/Cargo.toml --workspace
cargo deny --manifest-path crates/Cargo.toml check bans licenses sources
```

Workspace lints deny unsafe code and common escape hatches such as unwrap,
expect, panic, todo, unimplemented, dbg, and print macros. Do not work around
these with broad allows.

## Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runxhq/runx](https://github.com/runxhq/runx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
