---
trigger: always_on
description: SEMAPRAX is an agent-native systems programming language: **meaning in,
---

# SEMAPRAX agent guide

SEMAPRAX is an agent-native systems programming language: **meaning in,
verified machine code out**. Human-readable `.spx` source is the canonical Git
projection; the versioned semantic graph is the preferred agent interface.

This file contains repository operating invariants. The internal documentation
map and change protocol live in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md); the
module map lives only in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Subagent model selection

Prefer a cheaper, less capable model for delegated work whenever it can handle
that bounded task reliably. Use Luna for straightforward tests, documentation,
and mechanical changes; use Terra for implementation that needs more reasoning.
Reserve the strongest models for complex semantic work, difficult debugging, or
review where a cheaper model is insufficient. Give subagents concise, relevant
context and escalate only when needed. This preference applies across sessions
unless the user explicitly overrides it.

## Read order

Before changing semantics, read:

1. [RFC 0001](docs/RFC-0001.md) for the language and toolchain contract.
2. [Completion matrix](docs/COMPLETION-MATRIX.md) for affected product rows.
3. [Architecture](docs/ARCHITECTURE.md) for data flow and trust boundaries.
4. [Quality gates](docs/QUALITY-GATES.md) for required verification.
5. The versioned specification that owns the changed syntax, protocol, ABI,
   report, workspace route, or target profile.

The [development guide](docs/DEVELOPMENT.md#read-before-changing-semantics)
maps change areas to their additional required references. The
[roadmap](docs/ROADMAP.md) is sequencing, not a reduction of the full goal.

## Non-negotiable invariants

- A safe source program has equivalent checked behavior on every backend that
  claims to implement the admitted feature.
- Evaluation order is left to right. Lazy boolean operands execute only when
  required.
- Public declarations have persistent `@id` identities. Expression identities
  may be revision-scoped.
- Source formatting, graph JSON, Wasm bytes, diagnostics, semantic patches,
  and contracted generated artifacts are deterministic.
- Failed or stale semantic transactions leave authoritative source unchanged.
- A successful managed-workspace transaction publishes one complete immutable
  generation through `ACTIVE`. It does not rewrite original source files or
  grant atomic visibility to Git, editors, or arbitrary raw-path readers.
- Evidence capsules carry no authority. Evidence-gated routes acquire their
  ordinary lock/authority first, replay before staging or candidate creation,
  and let only the live invocation perform the final commit or `ACTIVE` pivot.
- Semantic impact and review are read-only and bound to exact source and patch
  bytes. Source drift fails closed.
- Capabilities are explicit. Compiler and generated code gain no ambient
  filesystem, process, network, home, secret, key, wallet, or signing authority.
- Ownership errors are compile-time diagnostics, never backend accidents.
- Cleanup inventory order is structural metadata. Cleanup-plan vectors are
  canonical runtime order and must never be sorted or repaired downstream.
- An owned call stages arguments left to right and transfers them together at
  its declared commit boundary.
- Failure selection is sticky. Cleanup cannot replace the selected status, and
  result publication follows postconditions and non-result cleanup.
- A settlement or concurrency model is proof data, not permission to perform a
  physical finalizer, spawn runtime work, or publish an artifact.
- No feature is “implemented” without the completion matrix's executable gate.

## Change protocol

1. Identify affected completion rows, invariants, and owning specifications.
2. Add a success case and stable diagnostic regression before or with the
   implementation.
3. When syntax carries runtime meaning, update parser, canonical formatter,
   resolver/HIR, verifier, semantic graph, native backend, and Wasm backend
   together.
4. Exercise both human and agent projections: canonical round-trip plus graph
   assertions.
5. Run `scripts/quality.sh full` on Unix, or reproduce the full profile in
   [Quality gates](docs/QUALITY-GATES.md).
6. Update architecture only for implementation ownership or trust-boundary
   changes, the matrix only for status/gate changes, the roadmap only for
   sequencing, and the changelog for history.

## Repository navigation

Writing or fixing `.spx` source starts with the compiler-checked
[agent quick reference](docs/AGENT-QUICK-REFERENCE.md): the admitted shapes,
the diagnostics that habits from other languages trigger, and their fixes, in
one page. Read the tour or an RFC only for a rule the reference does not state.

Use the repository's semantic tools for bounded questions about one declaration
rather than reconstructing SEMAPRAX meaning from source text:

```sh
cargo run --locked -p semaprax -- query <file>
cargo run --locked -p semaprax -- doc <file> [--json]
cargo run --locked -p semaprax -- context <file> <stable-id> --depth 1 --filters contracts,ownership --max-bytes 4096
cargo run --locked -p semaprax -- graph <file>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wavect/semaprax](https://github.com/wavect/semaprax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
