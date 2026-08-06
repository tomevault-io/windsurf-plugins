---
trigger: always_on
description: This file is for AI coding agents working in this repository. It captures the
---

# FrameForge Agent Guide

This file is for AI coding agents working in this repository. It captures the
standing project context, engineering constraints, validation rules, and
preferred workflows so new sessions can start without re-deriving them from
chat history.

The scope of this file is the whole repository.

## Project Goal

FrameForge is a hardware/software video-compression research workspace. The
repository contains:

- Rust software models for codec syntax and reconstruction.
- SystemVerilog RTL implementations of the same encoder subsets.
- Shared verification scripts that compare software, RTL, and external
  reference decoders.
- Shared synthesis/reporting infrastructure for rough Yosys estimates and
  optional Vivado timing/resource checks.

The current deliverable focus is a small but real encoder feature set that can
be validated end-to-end:

- VVC/H.266: 8-bit planar 4:2:0 lossy residual and 4:4:4 lossless
  screen-content subset.
- AV2: 8-bit planar 4:4:4 lossless screen-content path plus a maintained
  lossy 4:2:0 residual path.

FrameForge values spec-aligned, auditable syntax generation over opaque
payloads or trace-only reproduction.

## Read First

At the start of a session, inspect the current tree and read the relevant docs
before making assumptions:

```sh
git status --short
sed -n '1,220p' README.md
sed -n '1,220p' docs/project/feature-matrix.md
sed -n '1,220p' docs/rtl/hardware-interface.md
sed -n '1,220p' docs/rtl/architecture.md
sed -n '1,220p' docs/synthesis.md
sed -n '1,220p' docs/validation/targets.md
```

For AV2 work, also read:

```sh
sed -n '1,260p' docs/av2/roadmap.md
sed -n '1,220p' docs/av2/progress.md
```

For current measurements, read the codec-specific reports:

- `docs/av2/quality-bitrate.md`
- `docs/av2/output-utilization.md`
- `docs/av2/synthesis.md`
- `docs/vvc/quality-bitrate.md`
- `docs/vvc/output-utilization.md`
- `docs/vvc/synthesis.md`

Generated outputs live under `verification/generated/` and `synth/out/`; these
are not source-of-truth documents.

Current numeric validation, throughput, runtime, and synthesis-memory targets
are maintained in `docs/validation/targets.md`. Treat that file as the source
of truth for milestone acceptance thresholds.

Related operational docs:

- `docs/project/feature-matrix.md` for current VVC/AV2 feature status.
- `docs/validation/failure-triage.md` for failure-specific debug workflow.
- `docs/validation/reporting-workflow.md` for report and commit sequencing.
- `docs/validation/local-assets.md` for local manifest and media policy.
- `docs/rtl/architecture.md` for the shared RTL block map.

## Suggested Session Bootstrap

For a new AI session, a good first prompt is:

```text
Please read AGENTS.md first, then inspect git status and the relevant FrameForge
docs before changing code. Follow the repository validation, synthesis, report,
RTL style, and git rules from AGENTS.md.
```

If the session has a specific focus, append one line such as:

```text
Today we are working on AV2 RTL throughput optimization.
```

This is usually better than pasting long historical context. The agent should
derive the current state from committed docs, reports, and the working tree.

## Non-Negotiable Validation Rules

- A regression passes only when every selected vector passes.
- Software and RTL bitstreams must match byte-for-byte for implemented paths.
- Software, RTL, and reference-decoder reconstructions must match by checksum
  for implemented paths.
- VVC validation uses VTM as the external decoder.
- AV2 validation uses AVM/reference-decoder as a decode-only reference. Do not
  use AVM as a bitrate reference encoder in normal validation.
- If one test vector fails, stop treating that regression as passing. Use the
  failure to drive an audit and fix.
- Do not weaken validation criteria to make an incomplete implementation look
  green. Unsupported geometry or syntax can fail as an implementation bug.
- For lossless 4:4:4 paths, reconstruction should match the input and PSNR
  should be infinite.
- For lossy 4:2:0 paths, record PSNR and bitrate deltas.

When debugging a failure, audit the relevant code against the spec or reference
implementation first. Traces are useful evidence, but they should confirm the
audited cause rather than replace the audit.

## Feature Development Loop

For new codec features, use this order unless the user explicitly asks
otherwise:

1. Implement the software model first, with named syntax decisions and local
   reconstruction.
2. Define the feature's module boundary before adding RTL. New feature logic
   should live in a focused submodule under the relevant codec/block directory,
   with the codec top mostly instantiating and wiring it.
3. Validate software against the external reference decoder.
4. Implement the matching RTL inside the planned submodule boundary.
5. Run focused smoke validation.
6. Run the relevant full regression set.
7. Run synthesis when RTL changed.
8. Update reports with bitrate, output-utilization, and synthesis deltas.
9. Commit source changes first when practical, then commit report updates with
   the source SHA recorded in the reports.

Every new syntax path should be traceable in source comments or docs to the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrieldiego/frameforge](https://github.com/gabrieldiego/frameforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
