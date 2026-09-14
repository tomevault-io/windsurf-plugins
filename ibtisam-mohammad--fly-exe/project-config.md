---
trigger: always_on
description: Status: canonical project direction
---

# MaleCNS Virtual Fly — Agent Source of Truth

Status: canonical project direction  
Version: 1.17
Last evidence review: 2026-09-08
Last implementation audit: 2026-09-13
Applies to: this repository and every subdirectory

## 1. Agent bootstrap

Read this file before planning or changing the project. The project's target is:

> A scientifically honest, stochastic, MaleCNS-constrained embodied sensorimotor model of a representative adult male *Drosophila*.

It is **not** a recovered copy of the imaged fly, a complete biological emulation, or a digital twin. MaleCNS fixes much of the anatomical wiring. It does not fix the living dynamics, peripheral sensors, internal state, neuromuscular transformation, body, or environment.

Current first milestone:

> Closed-loop, flat-ground walking with limited sensory input, while preserving the MaleCNS brain–VNC–motor pathway and exposing every borrowed, fitted, or engineered parameter.

Agents must preserve these decisions unless the user explicitly changes the goal or a documented project decision supersedes them.

### Quick retrieval index

These stable keys are intended for agent search and handoff:

```text
PROJECT_GOAL: MaleCNS-constrained embodied adult-male sensorimotor model
CLAIM_BOUNDARY: population-plausible model; not source-fly recovery or digital twin
CANONICAL_CONNECTOME: MaleCNS v1.0
CURRENT_STAGE: Stage 2 fitted neural dynamics (active); the ADR-2026-006 evidence-chain repair is complete and V0 is reissued
DATA_STATUS: seven-artifact MaleCNS v1.0 flat-connectome profile checksum-locked; lossless contact derivative and independent dual-layout rebuilds validated
HIGHEST_VALIDATION_TIER: V0 Structural, reissued 2026-09-08 as bundle 20260908T060641Z_V0 under ADR-2026-006; unchanged by every DEMO-01 and DEMO-02 result, and the only tier names that exist are V0 to V8 in flysim.evidence.ValidationTier
ENGINEERING_STATUS: the Stage 2 exit gate is contract stage2-exit-gate-v4 and is 0 of 3 -- see STAGE2_EXIT_GATE below, which this line used to contradict by reporting the superseded v2 and its one passing leg; the uEPSC kernel is wrong in decay (0.463 against 0.30) and amplitude (13.6 pA median under), contact number is shown to move opposite to the published unitary-current scaling, the heterogeneous GeNN kernel is verified bit-identical to the homogeneous one on the full graph, and two registered MBON07 values are shown to depend on their measurement rule; NO tier is awarded anywhere, and the V1-limited and V2-restricted labels that appeared in four contracts and three documents were never members of the V0-to-V8 ladder and are removed (ADR-2026-018)
EON_SHOWCASE_STATUS: eon-showcase-v1 is retained as a reproducible limited preview but is withdrawn as the final public showcase under ADR-2026-020. Its frozen evaluator accepted 2 of 3 serial-story seeds, but it omitted its own 2.5 mm grooming gate; the hero records 8.477 mm. Navigation also uses a raw odour-gradient controller term and central relays, and two alleged causal controls are only serial-sequence dependency checks. The corrected evaluator now fails closed on a recorded behavioural violation. No tier changes
EON_SHOWCASE_V2_STATUS: eon-showcase-v2 is preregistered and blocked. It assembles three separately executed contracts: DEMO-01 visual-target approach at three frozen held-out targets, DEMO-02 grooming with G1-G6 including the 2.5 mm bout cap, and corrected MN9 feeding after a neural-only tarsal-taste operating-point search. Every component requires readout-ablated, stimulus-absent, command-replay and graph-free controller-only controls. The 60-90 s 1080p final cut must compare exact against ablation and say the chapters are separate. The validator is fail closed and awards no tier
EON_CINEMATIC_STATUS: the v1 1920x1080 H.264 cut remains checksum-locked as a historical limited preview, not the final showcase. Its simulation content is unchanged: it visibly distinguishes the 0.25 mm red food object from the engineered 1.0 mm thorax-proximity sucrose halo, but it inherits v1's bypasses and failed 8.477 mm grooming displacement. Video SHA-256 97a3790ed0ab6ffb2eca3c3e49f5f6807bda67cf1b6f489ed575252792c268b7
LIVE_MULTIFLY_STATUS: implementation candidate complete in the working tree under ADR-2026-021. The browser can place world stimuli but cannot write neural, motor or body state. Exact full-CNS states share one 25,563,197-edge connectivity allocation while retaining independent state for all 165,122 neurons. The 2.01-s dirty-tree engineering benchmark completed for 2 and 4 states; every state reduced distance to the calibrated target. Measured biological/wall rates were 0.167 and 0.080, so neither mode is real time. The artifact is /srv/flybrain-data/evidence/multifly/multifly-capacity-working-tree.json, SHA-256 8e64e3cecc00df9d9cec32e23208fa1619eb357edbd52a077462b12fc2b3d260. CPU preview is explicitly no-CNS; the shared collision-disc body is E; no tier, social-behaviour or cross-animal claim is awarded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ibtisam-Mohammad/Fly.exe](https://github.com/Ibtisam-Mohammad/Fly.exe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
