---
trigger: always_on
description: This repository contains **cryoROLE 2.0**, a policy-driven cryo-EM relative-orientation analysis platform.
---

# AGENTS.md

## Purpose

This repository contains **cryoROLE 2.0**, a policy-driven cryo-EM relative-orientation analysis platform.

The core workflow is:

```text
input -> preflight -> run -> inspect -> optional canonicalize -> explore -> confirm selection -> export
```

The primary engineering goal is to make this workflow stable, auditable, scalable, reproducible, and safe for large cryo-EM particle sets. Do not add new analysis variants before preserving the scientific and artifact contracts below.

---

## Required read order

For any behavior-changing work, read documents in this order:

1. `AGENTS.md` — non-negotiable guardrails and current priority.
2. `docs/cli_reference.md` — public commands and policies.
3. `docs/output_files.md` — artifact layout and scientific/display distinctions.
4. `docs/workflow_ux.md` — workflow and selection boundaries.

If a task changes persistence, CLI behavior, artifact layout, visualization, selection, export, or memory behavior, update the relevant document together with code and tests.

For behavior-changing offline animation work, read
`docs/animation_export.md` after the four documents above. It owns the proposed
animation-specific coordinate, rendering, artifact, and validation contract.

---

## Current active priority

The active engineering priority is:

```text
Keep the production-scale workflow safe while enforcing typed service boundaries.
```

Core Productionization is implemented. Workflow UX adds shared preflight,
artifact-derived status/next guidance, offline draft selection, and a resumable
guide without changing the scientific backend. The CLI is a frontend: input
policy resolution, source-identity verification, run orchestration,
canonicalization, visualization, selection, and Selection artifact writing
belong to shared typed services. NPZ visualize/select paths must filter or
evaluate compact arrays before materializing plotting/compatibility tables.

Canonicalization, visualization, selection, and export already have a working command-layer contract. Do not destabilize them while refactoring the run backend.

### Workflow UX

- `preflight` and `run --dry-run` must use the same input/matching service as production `run`.
- Preflight is side-effect-free and reports `READY`, `READY_WITH_WARNINGS`, or `BLOCKED`.
- Interactive display sampling and filters are display-only. Exact selection counts and Confirm operate on the full parent NPZ.
- Explore drafts are not Selection artifacts. Only explicit Confirm or `cryorole select` creates a scientific Selection.
- Interactive radius selection must reuse the Python SO(3) selection evaluator.
- The interactive server binds only to `127.0.0.1`, exposes no arbitrary filesystem routes, and uses no public CDN.
- `status` and `next` derive state from actual artifacts, manifests, and completion markers.
- `guide` must not silently enable row alignment, allow low overlap, canonicalize, select, export, or overwrite.
- `preflight`, `run`, and new-run `guide` must use the same resolved input-policy service; mapping-file policy must never fall back to a default identity key.
- Formal run consumption must revalidate source content identity, not only size and modification time.
- CLI select and interactive Confirm must use the same standard Selection artifact writer.

---

## Non-negotiable scientific invariants

1. Do not change the scientific definition of relative orientation:

```text
RO = R_ref^-1 R_mov
```

2. Do not silently change the passive-frame interpretation currently used by cryoROLE.
3. Keep source convention conversion centralized in the normalization layer.
4. Do not scatter `.T`, `.inv()`, active/passive conversion, or Euler convention logic across downstream modules.
5. Internal rotation truth is a `3 x 3` active rotation matrix. Rotation vectors, quaternions, and Euler angles are derived representations.
6. Euler angle output convention is a derived-representation policy. Public cryoROLE RO/RV Euler output uses extrinsic fixed-axis ZYX; intrinsic ZYX is legacy/internal compatibility only.
7. Do not conflate analysis-space quantities with display-space quantities.
8. Do not write display-only reparameterizations back into STAR/CS metadata.
9. Do not overwrite original input files.
10. Any behavior that changes interpretation must be controlled by an explicit policy and recorded in reports/manifests.

---

## Input and matching guardrails

### RELION `.star`

- RELION `Rot/Tilt/Psi` must be parsed as intrinsic `ZYZ` Euler angles.
- SciPy parsing must use uppercase `"ZYZ"`.
- RELION passive-to-active conversion must remain centralized and tested.
- Public `cryorole run` may default STAR identity matching to `_rlnTomoParticleName` first, then `_rlnImageName` / `rlnImageName`, only when the resolved column is present and safe.
- Never assume row-order correspondence between two STAR files unless the user explicitly requests row-aligned mode.
- Never use Euler angles, origin shifts, or other refinement-result columns as default particle identity keys.
- If RELION identity ambiguity remains unresolved, fail before RO computation.

### CryoSPARC `.cs`

- Treat `.cs` as a native input format.
- Default identity key is `uid`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yifancheng-ucsf/cryorole](https://github.com/yifancheng-ucsf/cryorole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
