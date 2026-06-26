---
trigger: always_on
description: Copyright 2026 Exochain Foundation
---

<!--
Copyright 2026 Exochain Foundation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at:

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

SPDX-License-Identifier: Apache-2.0
-->

# AGENTS.md — AI Development Instructions for EXOCHAIN

This document provides instructions for AI agents (Claude, Codex, Copilot, or any
LLM-based development tool) working on the EXOCHAIN constitutional trust fabric.

EXOCHAIN is a Rust workspace implementing a governance runtime where every operation
is constitutionally adjudicated. The system enforces separation of powers, consent-based
authority, and cryptographic provenance for all actions.

## Constitutional Constraints

These constraints are non-negotiable. Every change must satisfy all of them.

### 1. Absolute Determinism

The same input must always produce the same output, across runs, platforms, and time.

- **No floating-point arithmetic.** The workspace denies `clippy::float_arithmetic`,
  `clippy::float_cmp`, and `clippy::float_cmp_const`. Use integer arithmetic or
  fixed-point representations. If you need fractional values, use basis points (1/10000)
  or millibels.
- **BTreeMap only.** Never use `HashMap` or `HashSet`. These have non-deterministic
  iteration order. Use `BTreeMap` and `BTreeSet` from `std::collections`, or the
  `DeterministicMap` alias from `exo-core`.
- **Canonical serialization.** All data that gets hashed must be serialized via CBOR
  using the `ciborium` crate with sorted keys. Never hash JSON directly since key
  ordering is not guaranteed.
- **No system time.** Use the Hybrid Logical Clock (HLC) from `exo_core::hlc` for
  all timestamps. Never call `std::time::SystemTime::now()` or `Instant::now()` in
  production code.
- **No randomness in logic.** Randomness is only permitted for key generation
  (`ed25519-dalek` key pairs). All governance logic must be purely deterministic.

### 2. No Unsafe Code

The workspace sets `unsafe_code = "deny"`. Do not use `unsafe` blocks, `unsafe impl`,
or `unsafe fn`. If you believe unsafe is required, document the justification and
request a constitutional amendment through the governance process.

### 3. Error Handling

- Use `thiserror` for error type definitions. Every crate has an error module.
- Prefer `Result<T, CrateError>` return types. Avoid `unwrap()` and `expect()` in
  non-test code (both are set to `warn` level).
- Every error variant must carry enough context to diagnose the failure without
  access to the source code.

### 4. The Eight Constitutional Invariants

Every action in the system must satisfy these invariants, enforced by the kernel
in `exo-gatekeeper`:

1. **SeparationOfPowers** — No single actor may hold legislative + executive + judicial power.
2. **ConsentRequired** — Action denied without active bailment consent.
3. **NoSelfGrant** — An actor cannot expand its own permissions.
4. **HumanOverride** — Emergency human intervention must always be possible.
5. **KernelImmutability** — Kernel configuration cannot be modified after creation.
6. **AuthorityChainValid** — Authority chain must be valid and unbroken.
7. **QuorumLegitimate** — Quorum decisions must meet threshold requirements.
8. **ProvenanceVerifiable** — All actions must have verifiable provenance.

## Workspace Architecture

```
crates/
  exo-core/          Foundational types: HLC, crypto, BCTS, DID, Hash256
  exo-identity/      DID management, identity verification
  exo-consent/       Bailment consent engine (BCTS state machine)
  exo-authority/     Authority delegation, permission chains
  exo-gatekeeper/    Kernel, invariants, combinator algebra, holons, MCP, TEE
  exo-governance/    Legislative branch: proposals, voting, quorum
  exo-escalation/    Escalation paths, human override
  exo-legal/         Legal compliance, jurisdictional rules
  exo-dag/           Immutable causal DAG ledger
  exo-proofs/        Cryptographic proof generation and verification
  exo-api/           External API surface
  exo-gateway/       Gateway routing, rate limiting
  exo-tenant/        Multi-tenant isolation
  decision-forum/    Deliberative decision-making forum

governance/          Constitutional documents and council assessments
tools/
  codegen/           Crate scaffolding generator
  syntaxis/          Node registry and workflow code generator
  cross-impl-test/   Cross-implementation consistency testing
```

### Dependency Order

`exo-core` is the root. All crates depend on it. The dependency graph flows:

```
exo-core
  -> exo-identity, exo-consent, exo-authority, exo-dag, exo-proofs
    -> exo-gatekeeper (depends on most of the above)
      -> exo-governance, exo-escalation, exo-legal
        -> exo-tenant, exo-api, exo-gateway
          -> decision-forum
```

## DAG DB Runtime Adapter

This PR package keeps upstream `exochain/exochain` as the substrate and adds DAG

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exochain/exochain](https://github.com/exochain/exochain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
