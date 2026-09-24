---
trigger: always_on
description: handles, endpoint/antenna/receiver geometry, endpoint polarization tensors,
---

# Channel Architecture Guardrails

ADR-033 accepts the breaking replacement product identity `witwin.channel`,
the `witwin-channel` distribution, and the single `_channel` extension. The
checkout directory may retain its current name, but installed/runtime/build
identifiers must not retain the predecessor suffix or add a compatibility
alias. During the bounded migration, follow ADR-033 whenever an older name in
this file conflicts with that accepted target.

These repository instructions apply to every file under the checkout directory
`channel/` and
take precedence over the monorepo-level agent guide. Keep `AGENTS.md` and
`CLAUDE.md` identical. Architecture changes must update both files in the same
commit.

## Non-negotiable compute policy

`witwin.channel` has exactly one production compute backend: the
compiled native CUDA/RayD extension.

- Every production hot path must execute in a native CUDA kernel. This includes
  geometry evaluation, RF/material/scattering physics, sampling, path
  evaluation, reductions, and production JVP/VJP/backward math.
- Torch is the tensor-facing API and orchestration layer, not a production
  numerical backend. A Torch CUDA expression is still a Torch implementation
  and is forbidden for hot-path physics or geometry.
- CPU production computation and CPU fallback are forbidden. Missing CUDA,
  RayD capability, a native symbol, a supported SM, or an ABI-compatible
  extension must fail loudly before partial computation.
- Under ADR-035, RayD `TraceBackend::Auto` may select RayD-owned OptiX or
  RayD-owned pure-CUDA tracing during scene construction. OptiX is the
  preferred performance path; missing OptiX alone is not a missing Channel
  capability. This native implementation choice is not a Torch/CPU/Dr.Jit
  fallback, a second owner, a retry policy, or permission for reduced results.
  An operation unsupported by the selected RayD backend must fail its typed
  capability validation before that operation launches numerical work or
  exposes output.
- Never recover from a native failure by using Torch operations, NumPy, Python
  loops, CPU code, finite differences, legacy RayD/DrJit dispatch, a reduced
  algorithm, zero tensors, empty success results, or detached gradients.
- Production AD must call registered native forward/JVP/VJP/backward companions.
  Torch autograd may dispatch those companions but may not reconstruct the
  numerical operation. Finite-difference production derivatives are forbidden.
- Do not recompute geometry already owned by RayD in Python or Torch.
- Device data must remain resident through the compute pipeline except at the
  sole ADR-032-accepted new compact-cardinality allocation boundary. That owner
  may copy
  only audited integer count metadata to the host and explicitly synchronize
  the caller's current stream to allocate exact `O(K)` output. It may not run
  CPU/Torch physics or numerical selection, hide the transfer behind allocation
  or Boolean indexing, or become a fallback. Other hot-path `.cpu()`, `.numpy()`,
  `.tolist()`, scalar extraction, host iteration, implicit synchronization, or
  avoidable host/device copies remain forbidden. This is not a claim that the
  whole solve has only one D2H/synchronization: pre-existing observed boundaries
  remain measurable optimization debt and require named owners plus E2E,
  memory, throughput, and exactness evidence before any change.
- Under ADR-032, production Path and Deterministic result shapes and
  `max_num_paths` represent actual compact rows, not provisioned storage.
  `path_capacity_per_pair`, `diffraction_state_capacity`, capacity-shaped
  public Path/PathTable results, and ADR-031 `Qr` are not production public API
  or solver requirements. The measured depth-3 Munich reflection boundary may
  issue at most six 4-byte count D2H copies, 24 bytes total, and must report the
  copy/synchronization time.
- Accepted genuinely fixed-capacity operations retain one runtime
  `CapacityFailureState`: a contiguous CUDA `int32[1]` bitmask initialized
  asynchronously on the caller's current stream. Every participant receives
  and retains that same typed object/storage, atomically ORs its owned failure
  bit, publishes only inert outputs after failure, and never traps or returns a
  partial result. Compact output must likewise be all-or-nothing: exact complete
  `K` rows in stable order or no usable result. Capacity is never a silent
  truncation policy.
- `capacity_failure_terminal_check` is the unique runtime-owned terminal
  observer. It consumes that typed state once after all result sanitizers,
  launches on the caller's current CUDA stream, preserves the bitmask, and
  device-fails only when a bit is set. It must never read the state on the host,
  synchronize, allocate a result, sanitize payload, or gain an intermediate or
  duplicate owner. A transaction that uses it installs exactly one call after
  all result sanitizers; dormant experiments may not add a production caller.

Python and Torch may perform non-numerical boundary work: API validation,
typed-contract construction, dispatch, orchestration, row selection, structural
packing, metadata, and result assembly. CPU/Torch reference implementations are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [witwin-ai/witwin-channel](https://github.com/witwin-ai/witwin-channel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
