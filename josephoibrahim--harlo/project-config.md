---
trigger: always_on
description: provides memory and safety.
---

# Harlo v6.1-MOTOR — Claude Code Project Instructions

## What This Is

A biologically-architected AI memory and action system.
Rust hot path + Python orchestration + Elenchus verification +
Co-Evolution Spiral + Inhibition-default Motor Cortex.

Architecture: Two hemispheres (Association + Composition), one Bridge
with Amygdala, one Modulation Layer with Blood-Brain Barrier, one
Verification Engine (Elenchus GVR), one Inquiry Engine (DMN), and
one Motor Cortex with Basal Ganglia gating.

All state is local. Cloud models provide reasoning; your machine
provides memory and safety.

## Tech Stack

- Rust (crates/hippocampus/) — Association Engine hot path
- PyO3 + maturin — Rust ↔ Python FFI
- Python 3.12+ — everything else
- SQLite + sqlite-vec — 1-bit vector bitwise matching
- Click — CLI framework
- jsonschema — Blood-Brain Barrier validation
- systemd / launchd — socket activation (0W idle)
- pytest — Python tests
- cargo test — Rust tests

## Running Tests

```bash
cargo test -p hippocampus          # Rust unit tests
pytest tests/ -v                   # Python unit tests
pytest tests/test_integration/ -v  # Integration tests
```

> Build/env gotchas (cost a debugging cycle on 2026-06-19 — don't re-hit):
> - `maturin develop --release` RE-RESOLVES the unpinned `>=` deps in pyproject
>   and silently upgrades the ML stack (numpy/transformers/xgboost) → native
>   segfaults in the suite. Pin the ML stack (constraints file) or rebuild the
>   ext without re-resolving (`maturin build` + `pip install --no-deps wheel`).
> - Exported `PYTORCH_MPS_HIGH_WATERMARK_RATIO` ≤ ~1.0 makes torch raise
>   "invalid low watermark ratio" on model load (also breaks `harlo recall`).
> - `PYTHONOPTIMIZE=1` (`-O`) strips production `assert`s → assert-based tests
>   fail "DID NOT RAISE". Unset for test runs. `ruff` must be installed for lint.

## The 33 Inviolable Rules

### Biological Constraints (v3.0)

1.  0-WATT IDLE: OS socket activation. No while True. No sleep().
    Daemon exits when idle. 0W between sessions.

2.  ACTION POTENTIALS: Hippocampal vectors MUST be 1-bit boolean
    arrays (Sparse Distributed Representations). Bitwise XOR
    (Hamming distance) for search. No float32. No cosine similarity.

3.  RUST HOT PATH: Association Engine is Rust via PyO3.
    Cold start: <5ms. Hot recall: <2ms. No Python in hot path.

4.  LAZY DECAY: Timestamp math on retrieval only. No polling.
    strength = initial * e^(-lambda * dt) + sum(retrieval_boosts)
    dt in DAYS (Unix seconds / 86_400); lambda = 0.05/day → 13.9-day
    half-life. See docs/adr/0003-decay-units.md.

5.  APOPTOSIS: twin consolidate physically DELETEs traces below
    epsilon. Runs VACUUM. Database file size decreases.

6.  MERKLE TREES: Composition stages use Merkle Tree hashing.
    Partial branch O(log n). Not full-file SHA256 O(n).

7.  AMYGDALA: SAFETY/CONSENT resolutions = 1-shot permanent reflex.
    Skip GVR. Skip 10-rep curve. Instant compile to cerebellum.

8.  JSON BARRIER: jsonschema.validate(). Strip epigenetic_wash on
    write path. Mood ephemeral. Facts permanent. No XML. No regex.

9.  ALLOSTATIC LOAD: Token velocity + prompt frequency, plus OPTIONAL
    opt-in biometric signals via the biometric_barrier per ADR-0001
    (see docs/adr/0001-healthkit-allostatic.md). Biometric signals
    default OFF per data type and never enter the trace / reflex
    pipelines — they live in the Modulation Layer only. Samples older
    than the configured freshness window (default 5 min) cannot drive
    cognitive_state="RED". High = DEPLETED = refuse to wake System 2.

10. ANCHORS: SAFETY/CONSENT/KNOWLEDGE/CONSTITUTIONAL = gain 1.0
    ALWAYS. Structural. Returns 1.0 before evaluating receptor density.

### Elenchus Constraints (v4.0)

11. TRACE EXCLUSION: verify() NEVER receives reasoning trace.
    Parameter must be None or absent. BUILD FAILS if present.

12. VERIFIED-ONLY CONSOLIDATION: Only VERIFIED resolutions become
    reflexes. FIXABLE/SPEC_GAMED/UNPROVABLE never consolidated.
    BUILD FAILS if unverified resolution leaks to reflex cache.

13. MAX 3 GVR CYCLES: ADHD guard. After cycle 3, promote FIXABLE
    to UNPROVABLE. Loop MUST terminate.

14. INTENT PRESERVATION: Bridge checks output answers the original
    intent, not a reframed easier question.

15. SPEC-GAMING DETECTION: Correct answer to wrong question is the
    dominant failure mode. Detect it. Surface it. Never consolidate it.

16. UNPROVABLE IS DIGNIFIED: Carries metadata (reason, what_would_help,
    partial_progress). First-class state. Park with dignity.

17. BURST DEFERS, NOT SKIPS: Queue unverified outputs during burst.
    Run GVR on burst exit. Surface problems.

18. RED OVERRIDES EVERYTHING: No GVR. No injection. No inquiry.
    No motor. Full stop. Recovery menu.

### Inquiry Safeguards (v5.1-v5.2)

S1. APOPHENIA GUARD: Minimum evidence threshold per inquiry depth
    (5/8/15/25 independent observations). Alternative hypothesis
    required. Confidence disclosure mandatory.

S2. EPISTEMOLOGICAL BYPASS: Inquiry outputs verified for tone +
    boundaries, NOT objective truth. Self-reported traces bypass
    Elenchus ONLY when consumed by src/inquiry/ namespace.
    Composition namespace gets standard verification (DIRECTIONAL).

S3. RUPTURE & REPAIR: Rejection = permanent non-decaying trace

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JosephOIbrahim/Harlo](https://github.com/JosephOIbrahim/Harlo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
