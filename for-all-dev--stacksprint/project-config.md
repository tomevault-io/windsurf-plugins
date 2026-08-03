---
trigger: always_on
description: This file is the source of truth for any Claude Code session working in this
---

# CLAUDE.md — Verified Inference Stack (VIS) 90-Day Prototyping Sprint

This file is the source of truth for any Claude Code session working in this
repository. Read it fully before making changes. When instructions here
conflict with intuition or upstream examples, this file wins; if it conflicts
with a signed-off spec in `specs/`, the spec wins and this file must be updated.

## 1. Mission

Ship, in 90 days, a working prototype of an ML inference stack whose
**control plane is formally verified** and whose **data plane is formally
isolated**. Concretely: a quantized LLM serving requests end-to-end where the
scheduler, KV-cache allocator, request lifecycle, and memory admission logic
carry machine-checked proofs (Verus), the component architecture runs on seL4
with capability-enforced isolation, and the GPU sits outside the trusted
computing base behind IOMMU-enforced DMA boundaries.

We are NOT verifying: model weights, kernel numerics on GPU, the Python
tooling, or drivers. We ARE verifying: every line of code that decides who
gets memory, whose tokens go where, and what the GPU is allowed to touch.

**The primary deliverable is the SPECS.** Full stop. A complete, expert-signed,
machine-validated specification suite — invariants, transition systems,
refinement structure, and Verus contracts (`requires`/`ensures`/ghost state)
on every control-plane interface — is worth more than any individual green
proof. Proof bodies are deliberately allowed to be admitted (`sorry`-style
`assume`/`admit`): we treat proof completion as a token-spend problem that
future language models and automated proof search will discharge against our
contracts (vericoding / proof-carrying-code posture). Specs are the part that
requires convened human experts and cannot be regenerated later; proofs are
the part that gets cheaper every quarter.

**Definition of success (day 90):** (a) a demo where a small quantized model
serves concurrent streaming requests on the seL4 image under QEMU and on one
reference ARM board; (b) 100% of control-plane interfaces carry signed-off
formal contracts traceable to `specs/invariants.md`, with every spec
*validated* (model-checked, mutation-tested, non-vacuous — see §2.2) even
where its Verus proof is admitted; (c) a proof-debt ledger enumerating every
admitted proof with its contract, ready to hand to automated proof
engineering; (d) the TCB audit document enumerating every trusted line and
every admit. Green proofs are recorded as a burndown metric, not a gate —
except the small "load-bearing set" defined in §2.4.

## 2. Non-negotiable principles

1. **The spec is the deliverable.** Code without its *contract* does not
   merge. Code with a contract but an admitted proof body merges freely —
   that is proof debt, and proof debt is a feature of this sprint, not a
   failure. "Unspecified" is the state that does not exist in this repo.
2. **Admitted ≠ unvalidated.** A `sorry` is only allowed against a spec that
   has been independently validated, because an admitted proof can silently
   hide a vacuous, unsatisfiable, or wrong spec. Validation means all of:
   (i) the corresponding TLA+/Z3 transition system model-checks the same
   invariant (this is cheap and catches most spec bugs); (ii) the spec is
   *satisfiable* — at least one concrete execution trace witnesses it;
   (iii) it kills its assigned mutants at the model level (§8.6). Expert
   sign-off happens on the spec, and only on a validated spec.
3. **Spec before code, spec survives code.** New components start as a TLA+
   or Z3 transition system in `specs/`, get validated per 2.2, get signed
   off, then get a Verus skeleton whose contracts are traceable (by ID: I1,
   I2, …) to the spec. Contracts are written to be *provable by a future
   agent with no context*: self-contained, no reliance on tribal knowledge,
   every ghost abstraction documented at its definition. Assume the entity
   discharging the proof has this repo and nothing else.
4. **A small load-bearing set stays green.** Not everything can be deferred:
   proofs whose failure would invalidate the *architecture* (the kv-alloc
   no-alias/refcount core, the scheduler admission bound, the validator's
   DMA bounds checks) are gate-blocking and must be fully discharged in the
   sprint. The review board owns this list in `specs/invariants.md`; it
   should stay under ~15 invariants. Everything else may be admitted.
5. **Honest debt accounting.** Every admitted proof, every
   `#[verifier::external_body]`, every `assume`, every axiomatized FFI
   boundary gets an entry in `docs/proof-debt.md` or `docs/tcb-ledger.md`
   (debt = we intend to prove it; TCB = we intend to trust it — never
   confuse the two) with an owner, the contract it owes, and a difficulty
   estimate. CI fails if the ledgers and code disagree (`tools/tcb-audit`).
6. **The GPU is an adversary.** Anything returning from the data plane is
   validated by contract-carrying code: shapes, ranges, buffer bounds,
   sequence tags. No control-plane component ever branches on unvalidated
   device data. (Validator contracts are in the load-bearing set.)
7. **Prototype honestly.** Cut scope, not spec rigor. A smaller fully
   specified system beats a larger loosely specified one. When in doubt,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [for-all-dev/stacksprint](https://github.com/for-all-dev/stacksprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
