---
trigger: always_on
description: WorldForge is a Python integration layer for testable physical-AI world-model workflows: provider
---

# Agent Guide

## Project Identity

WorldForge is a Python integration layer for testable physical-AI world-model workflows: provider
adapters, world state, planning, evaluation, benchmarking, diagnostics, and host-owned optional
runtimes. It is a library and CLI, not a hosted service, on-chain contract, or end-user
application.

Intended users are Python developers building provider adapters, local world-model experiments,
evaluation harnesses, and testable prototypes.

## Architecture Map

- `src/worldforge/models.py`: domain models, serialization helpers, validation errors, request
  policies, provider metadata, media/result types, and structured planning goals.
- `src/worldforge/capabilities/__init__.py`: runtime-checkable capability protocols for narrow
  `Cost`, `Policy`, `Generator`, `Predictor`, `Reasoner`, `Embedder`, `Transferer`, and
  `RunnableModel` integrations.
- `src/worldforge/framework.py`: `WorldForge`, `World`, persistence, planning, prediction,
  comparison, diagnostics, and top-level evaluation helpers.
- `src/worldforge/providers/base.py`: provider interfaces, `ProviderError`, remote-provider
  base behavior, and `PredictionPayload`.
- `src/worldforge/providers/observable.py`: internal wrapper that adds `ProviderEvent`, health,
  profile, info, and timing behavior around pure capability protocol implementations.
- `src/worldforge/providers/catalog.py`: provider factories and auto-registration policy for the
  in-repo provider catalog.
- `src/worldforge/providers/mock.py`: deterministic local provider used by tests, examples, and
  contract checks.
- `src/worldforge/providers/cosmos.py` and `runway.py`: real HTTP adapters with typed timeout,
  retry, polling, download policies, and response parsers.
- `src/worldforge/providers/leworldmodel.py`: real optional LeWorldModel JEPA cost-model adapter
  for scoring action candidates through `stable_worldmodel.policy.AutoCostModel`.
- `src/worldforge/providers/gr00t.py`: experimental host-owned NVIDIA Isaac GR00T PolicyClient
  adapter for selecting embodied action chunks through the `policy` capability.
- `src/worldforge/providers/lerobot.py`: host-owned Hugging Face LeRobot `PreTrainedPolicy`
  adapter for selecting embodied action chunks through the `policy` capability.
- `src/worldforge/providers/jepa_wms.py`: candidate contract scaffold for
  `facebookresearch/jepa-wms` score-provider work; it supports injected test/runtime scoring and a
  host-owned torch-hub runtime but is intentionally not exported or registered.
- `src/worldforge/providers/remote.py`: credential-gated scaffold providers for `jepa` and
  `genie`; these intentionally use deterministic mock behavior after credential checks.
- `src/worldforge/evaluation/`: built-in generation, physics, planning, reasoning, and transfer
  suites plus report renderers.
- `src/worldforge/benchmark.py`: capability-aware provider latency, retry, and throughput harness.
- `src/worldforge/observability.py`: composable `ProviderEvent` sinks for JSON logging, in-memory
  recording, and metrics aggregation.
- `src/worldforge/rerun.py`: optional Rerun SDK bridge for sanitized provider events, world
  snapshots, plans, benchmark reports, robotics showcase visual layers, and JSON artifacts. Rerun
  is not a provider capability and stays behind the `rerun` extra or host-owned optional runtimes
  that already provide `rerun-sdk`.
- `src/worldforge/testing/`: reusable adapter contract helpers.
- `src/worldforge/demos/`: packaged demo entry points exposed through `uv run` console scripts.
- `src/worldforge/demos/lerobot_e2e.py`: packaged LeRobot policy-plus-score planning demo exposed
  through `uv run worldforge-demo-lerobot`.
- `src/worldforge/demos/rerun_showcase.py`: packaged Rerun observability and artifact showcase
  exposed through `uv run --extra rerun worldforge-demo-rerun`.
- `src/worldforge/harness/`: optional TheWorldHarness TUI package. Keep flow metadata and runners
  independent from Textual; `tui.py` is the only Textual-dependent module. Current flows cover
  LeWorldModel score planning, LeRobot policy-plus-score planning, and provider diagnostics plus
  benchmark comparison.
- `src/worldforge/smoke/`: packaged optional-runtime smoke entry points exposed through `uv run`
  console scripts.
- `src/worldforge/smoke/lerobot_leworldmodel.py`: optional host-owned real robotics showcase that
  composes a LeRobot policy checkpoint with a LeWorldModel score checkpoint through
  `World.plan(..., planning_mode="policy+score")`.
- `src/worldforge/smoke/robotics_showcase.py`: one-command PushT real robotics showcase that wires
  the packaged PushT observation, score, translator, and candidate bridge defaults into
  `lewm-lerobot-real`.
- `src/worldforge/smoke/pusht_showcase_inputs.py`: packaged PushT showcase hooks for building the
  LeRobot observation, LeWorldModel score tensors, and checkpoint-native action candidates.
- `src/worldforge/smoke/leworldmodel_checkpoint.py`: optional host-owned builder for creating the
  LeWorldModel `*_object.ckpt` file expected by `AutoCostModel` from Hugging Face LeWM assets.
- `examples/leworldmodel_e2e_demo.py`: checkout-safe end-to-end LeWorldModel provider-surface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbdelStark/worldforge](https://github.com/AbdelStark/worldforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
