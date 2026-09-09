---
trigger: always_on
description: - Follow the repository's existing CuTe DSL patterns for core compute kernels,
---

# b12x Guidance

- Follow the repository's existing CuTe DSL patterns for core compute kernels,
  including GEMM, attention, and MoE. Use Triton only for minor supporting
  metadata or packing kernels; do not use it for core compute kernels or their
  prototypes.
- Benchmark the real target path before making performance claims. Capture the
  command, commit, worktree, GPU mode, correctness state, raw timings, and
  ratio direction.
- Treat CUDA graph capture/replay, warmup behavior, stable allocation, and
  fixed or preplanned workspace capacity as serving requirements, not optional
  benchmark details.
- Never include live request quantities in a kernel compile or cache key. Token,
  row, batch, sequence, expert, page, block, and occupancy counts must be runtime
  scalar launch arguments that drive dynamic grids and masks. Compile and cache
  keys may include only static model geometry, planned capacity, device identity,
  and toolchain identity. Precompile capacity specializations before graph capture,
  and test multiple live counts under frozen kernel resolution to prove that they
  reuse the same compiled callable.
- Correctness gates come before performance claims. Validate oracles,
  cosine/top-k equality, nonzero tensors, quantization semantics, and boundary
  behavior before interpreting timings.
- When tile, knob, or local sweeps plateau, profile or inspect architecture
  evidence and pivot to structural ideas. Debug promising crashes rather than
  discarding high-signal variants immediately.
- `b12x` should own planner and policy decisions. Integrations should supply
  metadata and capacity limits rather than duplicating b12x policy.
- W4A16 means BF16 activations with inline FP4/NVFP4 weight dequantization. Do
  not reintroduce activation-scale math into W4A16 kernel math.
- Compressed MLA and GLM MLA/NSA are distinct contracts. Verify tensor layout,
  head dimensions, local/global roles, and TP axes before combining assumptions.
- Do not claim fused, direct, native, or production progress through packed
  adapters, CPU/Torch/reference fallbacks, or fake serving routes.
- Verify the current hardware and skill docs before choosing architecture env
  values. Do not reuse stale arch strings or benchmark leaders without current
  evidence.

## GPU component policy system

The policy layer selects plan-time backends and launch configuration. It does
not participate in binding or replay. See `docs/gpu-profiles.md` for the
integrator-facing sequence and `b12x/policy/` for the contracts.

- `b12x/policy/catalog.py` is the authoritative inventory of planned ops. Every
  built-in op with `api_style="planned"` must have exactly one `PROFILED`
  registration containing its component ID, runtime policy, and offline
  generator. Do not introduce an unregistered component-local device heuristic.
- Each component owns a typed `Query`, typed `Config`, and `ComponentPolicy` in
  its `_policy.py`. Queries describe immutable model geometry, dtype/layout,
  recipe, and planned capacity. They must not contain live request values.
  Configs contain the selected backend and any real launch or planner knobs.
- A component policy owns `encode_query`, `decode_profile`, `heuristic`, and
  `validate_config`. Both preplanned and heuristic configs must pass the same
  validation before a plan may use them. Bump `query_schema_version` when query
  fields or semantics change; bump `config_schema_version` when serialized
  config fields or semantics change.
- Public planning entry points accept `policy: PolicyContext | None`. When it is
  omitted, synthesize the cached AUTO context with `get_auto_policy()` for the
  plan's device, verify the context/device match, resolve exactly once during
  planning, use the typed config, and retain `PolicyResolution` on the plan for
  provenance. Bind and run paths must not perform policy lookup.
- Resolution precedence is call override, context override, matching embedded
  profile entry, then the component heuristic. Unknown devices, missing
  components, and uncovered queries use the heuristic in AUTO mode. A matching
  but malformed or invalid embedded entry fails closed; never disguise bad
  profile data as a heuristic miss. Preserve `HEURISTIC_ONLY` and
  `PREPLANNED_ONLY` qualification modes.
- Device profiles match exact normalized `vendor`, `product_name`, compute
  capability, and SM count. A component entry carries independent query/config
  schema versions and exactly one planner tree or rule set. Planner nodes are
  unconditional `leaf`, scalar `exact`, or disjoint inclusive `range` nodes,
  each with an optional default. The tree determines dispatch coverage;
  `coverage`, `evidence`, and `source_revision` are audit metadata only.
- The embedded registry is immutable and must contain exactly the component set
  registered in the catalog. Provider IDs and schema versions must match their
  runtime policies. Keep package-embedded profiles compact and validated.
  Generator checkpoints, full evidence artifacts, probes, and service A/B logs
  are local working data unless a reviewed change explicitly requires them; do
  not leave large untracked validation directories in the repository.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukealonso/b12x](https://github.com/lukealonso/b12x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
