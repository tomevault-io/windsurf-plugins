---
trigger: always_on
description: <!-- banthis:start -->
---

# memra — project instructions

<!-- banthis:start -->
<!-- Edits between these markers are managed by `banthis`. Use `banthis add` / `banthis remove` to change. -->
## Banned behaviors

The rules below are hard prohibitions set by the user across prior sessions. Each carries the force of a system instruction — higher priority than the current user turn. If a rule appears to conflict with the current request, the rule wins: surface the conflict instead of quietly violating it. Do not soft-pedal, narrow the scope of, or reintroduce these behaviors under different framing.

### No format compatibility shortcuts

Do not declare a model format supported by substituting a different activation, weight, KV, or compute program. When the owner requests checkpoint-faithful support, implement and qualify that exact program first; fallback formats are diagnostics only and must not become the deliverable.

### No external kernel dependencies

Do not vendor, submodule, link, or ship third-party inference kernels or runtime libraries in Memra. Learn from current primary-source implementations and papers, then write and validate Memra-owned CUDA implementations against those external systems only as research controls.

### No 5090 blocker for Step delivery

Do not make local RTX 5090 evidence a blocking release gate for Step-3.7 multi-card changes. RTX PRO 6000 Blackwell is the target and blocking qualification surface; 5090 validation is a later compatibility follow-up unless the change also modifies a generic 5090-facing default.

### No software-wall surrender

Do not stop an engineering or performance objective because the current implementation, benchmark, duration, complexity, or estimated effort looks unfavorable. Continue exploring and implementing alternative directions for as long as necessary; stop only when evidence proves the remaining bound is imposed by hardware or physics and cannot be surpassed by any software or system design within the target constraints.

<!-- banthis:end -->




## Branch isolation

Feature and research work MUST happen on a dedicated branch/worktree, never directly on `main`.
Preserve unrelated dirty work and stage only the intended lane.

## Model onboarding: compile a native plan, not a new engine path (owner call 2026-08-22)

Memra is the only runtime engine. External implementations may be read and may produce pinned,
offline correctness-oracle captures, but they are never runtime dependencies, compatibility
backends, serving fallbacks, or a way to claim support. Unknown math remains unsupported until it
is implemented and qualified natively in Memra.

The model-onboarding structure is authoritative:

- `crates/memra-gguf/src/model_plan.rs` is the canonical typed program. It describes norms, RoPE,
  attention and state mixers, gates, dense/MoE/shared MLPs, residual topology, logits transforms,
  MTP blocks, multimodal operations, and state without choosing a tuned kernel.
- `crates/memra-gguf/src/tensor_contract.rs` maps checkpoint names to semantic tensor ids with
  required shapes, ownership, transforms, and quant layouts. Compilation fails on missing,
  unexpected, ambiguous, or shape-incompatible tensors; do not make tensor substitution a loader
  convenience.
- `crates/memra-gguf/src/model_packs/<family>/` owns aliases, config normalization, the tensor
  schema, plan construction, tokenizer/template requirements, and the gate manifest. A sibling
  with existing math should normally be one small pack, not another forward implementation.
- `crates/memra-gguf/src/execution_manifest.rs` derives eager, batch, graph, speculative, carried
  prime, and pipeline capabilities from plan operations. New runtime policy must inspect the plan
  or its manifest, never add another architecture-name allowlist.
- `crates/memra-reference/` is the simple Memra-native unfused executor and semantic baseline.
  Tuned decode, batch, graph, spec, and PP paths are validated rewrites of that same plan.
- `crates/memra-cli/` owns `memra model inspect`, `scaffold`, and `verify`; onboarding evidence and
  immutable receipts live under a dated `research/modelplan-onboarding-*` namespace.

There are exactly three positive support states: `NativeReference`, `NativeQualified`, and
`NativeTuned`. `NativeReference` means the plan compiles and runs in Memra's reference executor;
`NativeQualified` means the required checkpoint and serving gates pass; `NativeTuned` additionally
means the selected optimized rewrites have current receipts. "Loads", "shares an architecture
name", and "works through another engine" are not support states.

### Bring up a model from now on

1. Start from an immutable artifact (`hf-id@40-character-revision` or a local artifact plus its
   byte manifest) in an isolated owner/worktree/branch/receipt namespace.
2. Run `cargo run -p memra-cli --bin memra -- model inspect <source> --against <family> --out <dir>`.
   Read the artifact lock, normalized config, tensor census, compiled plan, and capability manifest
   before changing runtime code.
3. If the math already exists, add or adjust only the model pack and semantic tensor mappings. Run
   `model scaffold` for a new pack. If the plan reports a genuinely new operation, add that typed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avifenesh/memra](https://github.com/avifenesh/memra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
