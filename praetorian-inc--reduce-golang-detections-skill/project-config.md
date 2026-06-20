---
trigger: always_on
description: Use when reducing VirusTotal/EDR detection rates on compiled binaries — systematic A/B testing methodology with comprehensive PE structural analysis for identifying and eliminating ML classifier signals
---


# Reduce EDR Detections

**Systematic methodology for reducing VirusTotal and EDR detection rates on compiled binaries through comprehensive structural analysis, iterative A/B testing, and ML feature vector optimization.**

## When to Use

- VT detection rate is too high on compiled binaries
- ML classifiers (Wacatac, MalwareX-gen, ML.Attribute, Evo-gen, etc.) are flagging output
- Need to identify which binary attributes trigger detection
- After changes that may alter the PE/ELF structure
- Preparing binaries for deployment

## Prerequisites

- VT API key at `/path/to/vt_apikey`
- `pefile` and `lief` Python libraries — preferably in a venv (`python3 -m venv .venv && . .venv/bin/activate && pip install pefile lief`). On PEP 668 systems where you can't use a venv, add `--break-system-packages` to a system-wide `pip install`.
- PE Structural Analyzer script: see `references/pe-structural-analyzer.md`
- A vanilla binary from the same toolchain as your target (e.g., `GOOS=windows GOARCH=amd64 go build`)

## Core Principles

1. **Triage detection type before choosing remediation.** A YARA-style verdict (`Trojan/Win.Sliver.R774471`) and an ML verdict (`Wacatac.B!ml`, `ML.Attribute.HighConfidence`) require fundamentally different fixes. YARA matches fixed bytes — rename strings, swap imports, restructure sections. ML is a statistical classifier — renaming strings cannot defeat it, and trying often makes things worse. Label every hit before starting.
2. **Change ONE variable per experiment. 10–20 samples per test. Build control and variant in the same VT upload window.** ML models (especially Microsoft Wacatac.B!ml) retrain on approximately a daily cadence. A control batch built today and a variant built tomorrow is not a valid A/B test — half the observed delta will be model drift.
3. **Don't fight the toolchain identity.** Making a Go binary look less like Go creates inconsistencies that _increase_ detection — including renaming natural internal type names (e.g. `Allocator`, `Preamble`) that show up slightly over-represented in detected samples. That's likely `strings -n 6` extraction noise, not a real signal. See `references/experiment-categories.md`.
4. **Validate `strings -n 6` tokens against source before acting on them.** The `strings` tool glues adjacent in-memory strings together, producing tokens that look like meaningful symbols but are two unrelated strings concatenated across a buffer boundary. Grep the actual source tree to confirm before making a suspicious token a hypothesis.
5. **Camouflage, not concealment.** The goal is to give the classifier a believable answer to "what is this binary?" Mimicking the gopclntab symbol fingerprint of a single large real Go project (ghost profiling) consistently outperforms stripping, obfuscating, or padding. One coherent project; blending multiple produces a binary that matches no known software.
6. **VT is not ground truth, and there is an irreducible floor.** Microsoft's cloud ML is substantially more aggressive than the local Defender engine. A binary at 100% Wacatac on VirusTotal can be clean on a real endpoint. Once detection drops to roughly 15–25% (the stochastic floor near the ML threshold), further structural optimization rarely pays back.
7. **Measure everything.** Use the full PE structural analyzer before and after each change. Features you don't measure can't be correlated with detections.
8. **Compare against vanilla.** Always compare your binary against a clean vanilla binary from the same toolchain. The delta between them is your ML signal.
9. **ML classifiers use feature vectors, not individual features.** Compound anomalies accumulate — fix the ones that diverge most from the vanilla baseline.
10. **On-sensor EDR models are purely additive.** Reverse engineering of a major EDR's on-sensor ML model revealed 20 gradient-boosted trees with 1,000 binary features, ALL leaf weights positive (0.05–2.25) — the model only penalizes, never rewards. Zero triggered features = score 0.0 = always passes. There is no "benign bonus" for looking legitimate — only penalties for looking malicious. Fewer anomalies = lower score = less detection.

## On-Sensor EDR Model Intelligence

Reverse engineering of a major EDR vendor's kernel driver and on-sensor ML model produced verified intelligence about how the static analysis pipeline works. This informs which PE features to prioritize.

### Model Architecture

- **20 gradient-boosted decision trees**, 8,976 total binary-feature nodes
- **1,000-dimensional binary feature vector** (present/absent per indicator)
- **Purely additive scoring** — all 514 non-trivial leaf weights are positive (0.05–2.25)
- **Binary feature model** — features are primarily binary (is indicator X set?), though ~380 float64 split values exist in the model data for computed features like entropy scores
- 2 sub-models (likely benign vs malicious binary classifiers)

### EDR Static Analysis Passes (Verified)

The EDR kernel driver runs these passes on every file write / process creation:

| Pass | Codes | Feature range | What it checks |
|------|-------|--------------|----------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praetorian-inc/reduce-golang-detections-skill](https://github.com/praetorian-inc/reduce-golang-detections-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
