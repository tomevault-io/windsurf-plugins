---
trigger: always_on
description: This file is for coding agents working in this repository. Follow it unless a more specific user instruction or a nested
---

# AGENTS.md - Fluentbase Development Guide

This file is for coding agents working in this repository. Follow it unless a more specific user instruction or a nested
`AGENTS.md` overrides it.

## Project Snapshot

Fluentbase is a Rust workspace for the Fluent L2 execution stack. The core idea is blended execution: EVM/SVM/WASM/UST
compatibility layers converge into rWasm IR and a single proof-friendly runtime/STF.

Important areas:

- `bins/` - binary entrypoints, especially the `fluent` CLI.
- `crates/` - core Rust crates (`runtime`, `revm`, `evm`, `sdk`, `codec`, `node`, `genesis`, etc.).
- `contracts/` - system contracts and genesis/runtime upgrade contract artifacts.
- `examples/` - example contracts/apps.
- `e2e/` - end-to-end tests and benchmarks.
- `evm-e2e/` - separate EVM state-test/fixture runner crate, intentionally excluded from the root workspace.
- `flips/`, `docs/` - design and documentation.

SVM-related crates are currently unstable and excluded from the top-level workspace unless explicitly requested.

## Working Rules

- Protect local work. Do not overwrite, reset, rebase, or delete user changes unless explicitly asked.
- Check `git status --short` before editing and again before reporting completion.
- Keep changes focused and minimal. Avoid opportunistic refactors.
- Prefer fixing root causes over broad compatibility shims.
- Do not vendor generated output or large artifacts unless the task explicitly requires it.
- Preserve `no_std` constraints where crates are configured for it.
- Be careful with genesis/runtime changes: they may be chain-breaking and require release/upgrade planning.

## Security Policy

This project maintains a security policy in [SECURITY.md](./SECURITY.md).
All AI coding agents must read and follow `SECURITY.md` before making security-sensitive changes.
It covers:

- Vulnerability reporting procedures
- Supported branches for security reports
- Threat model and trust boundaries
- Secure coding guidelines for Fluentbase runtime and contract work
- Constraints for AI coding agents

Do not duplicate the full security rules here; keep `SECURITY.md` as the single source of truth.

## Security Audit Mode

When asked to audit for vulnerabilities, run this playbook. It is tuned for high-capability models
(e.g. Fable 5) that follow instructions literally and under-use parallelism unless told to fan out.

Operating rules:

- Report EVERY finding with severity + confidence. Do NOT pre-filter for importance or certainty —
  coverage is the goal; triage is a separate downstream pass. Instructions like "be conservative" or
  "only high-severity" cause real bugs to be silently dropped; do not apply them in audit mode.
- Take the full scope up front and run at `high`/`xhigh` effort. Do not narrow scope on your own.
- Fan out subagents across the independent crate groups below, in parallel. Each subagent must read
  real source — never reason from filenames or guess.
- Trace data flow to a sink. A category name ("possible overflow") is not a finding; "guest length
  `L` reaches `copy_from_slice` at `file:line` with no bounds check, panicking the runtime" is.

Priority order (consensus-critical first): memory safety in `unsafe` → integer overflow in gas/fuel
→ missing bounds checks in decoders → panics reachable from untrusted input (DoS / chain halt) →
nondeterminism (consensus split) → host↔guest isolation breaks.

Fan-outgroups (independent — audit concurrently):

| Subagent           | Scope                                                                                | Primary risk                                                                     |
|--------------------|--------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| runtime            | `crates/runtime`                                                                     | syscall handlers reading guest memory, fuel-charge ordering, host↔guest boundary |
| codec              | `crates/codec`, `crates/codec-derive`                                                | header-vs-body bounds (see SECURITY.md checklist), unbounded allocation          |
| evm/revm           | `crates/evm`, `crates/revm`                                                          | gas-arithmetic overflow, EVM-semantics divergence                                |
| crypto/precompiles | `crates/crypto`, `contracts/*`                                                       | precompile input-length validation, modexp gas, curve membership                 |
| sdk                | `crates/sdk`                                                                         | unsafe allocator, LEB128 decode, storage key/index math                          |
| types/build/ci     | `crates/types`, `crates/build`, `crates/genesis`, `crates/node`, `.github/workflows` | bincode allocation, WASM validation, CI script injection                         |

Per-finding reporting contract: severity (critical/high/medium/low) + confidence; `file_path:line`;
the data path from untrusted input to sink; a concrete exploit scenario; the fix.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluentlabs-xyz/fluentbase](https://github.com/fluentlabs-xyz/fluentbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
