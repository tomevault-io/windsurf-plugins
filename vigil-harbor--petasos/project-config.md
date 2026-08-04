---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Petasos is a pluggable, session-aware content security pipeline for Python AI agents. It composes OSS scanners (LLM Guard, LlamaFirewall, Presidio) behind a unified `Scanner` protocol, adds session-aware orchestration (frequency tracking, escalation tiers, profile-driven tuning, tool call guard), and exposes every configuration surface for frontend binding. Published on PyPI as `petasos`.

Primary consumer: Hermes Agent (Nous Research, Python 3.11+). Petasos imports in-process as a Python library — no sidecar, no REST, no subprocess.

### Two Consumer Platforms

Hermes Desktop ships on macOS (native) and Windows (.exe / Electron). The macOS integration path is well-documented in the wiki. The Windows path has platform-specific footguns. Key differences: file tools bypass the terminal sandbox, hooks spawn via Git Bash (MINGW64) adding 100-200ms, config sections get wiped by the UI model switcher, and process signal handling diverges. PET-6 through PET-11 must account for both platforms.

Petasos is [Drawbridge](https://github.com/ziomancer/clawmoat-drawbridge)-*inspired* (prior art) but fully uncoupled: own repo, own ticket prefix (PET), own release cadence, own threat model. No shared rule package, no cross-runtime conformance with Drawbridge.

## Status

Work items tracked in Plane (PET project). The file `petasos-project-spec-DRAFT-PARKED.md` is a superseded earlier draft preserved for reference — it proposed Drawbridge coupling that was explicitly rejected.

## Build & Run

```bash
pip install -e .                    # base install (no ML deps)
pip install -e ".[llm-guard]"       # + LLM Guard scanner
pip install -e ".[llamafirewall]"   # + LlamaFirewall scanner
pip install -e ".[presidio]"        # + Presidio PII scanner
pip install -e ".[all]"             # all scanner backends

# Tooling
ruff check .                        # lint
ruff format .                       # format
mypy --strict .                     # type check
pytest                              # run all tests
pytest tests/test_minimal_scanner.py # single test file
pytest -k "test_normalize"          # single test by name
pytest --cov                        # coverage report
```

Build backend is Hatch (`pyproject.toml`).

## Architecture

### All Features Free

All features ship free and keyless. Detection (scanner protocol, pluggable backends, syntactic pre-filter, normalization, PII anonymization) and session intelligence (frequency tracking, escalation tiers, profiles, tool call guard, audit trails, alerting) are both available out of the box. License machinery (`LicenseValidator`, `activate`/`deactivate`) is preserved for future supporter-token or compliance-tier use but does not gate any feature.

### Scanner Protocol

The load-bearing abstraction. Every detection backend implements:

```python
class Scanner(Protocol):
    @property
    def name(self) -> str: ...
    async def scan(self, text: str, *, direction: Direction = "inbound",
                   session_id: str | None = None) -> ScanResult: ...
```

Four backends: `MinimalScanner` (23 rules across 6 families, zero deps, always ships), `LlmGuardScanner` (extras), `LlamaFirewallScanner` (extras), `PresidioScanner` (extras).

### Pipeline Stages

```
Input → Normalize (NFKC, zero-width, homoglyph, RTL)
  → Syntactic pre-filter (23 rules, always runs)
  → Fan-out to N scanners (asyncio.gather)
  → Merge findings (dedup overlapping positions)
  → Frequency update → Escalation check
  → Anonymize (if PII + enabled)
  → Audit → Alerting
  → PipelineResult
```

### Target Layout

```
petasos/
├── __init__.py
├── _types.py            # Scanner protocol, ScanResult, ScanFinding, Direction, etc.
├── normalize.py
├── pipeline.py          # Pipeline class — central orchestrator
├── config.py            # PetasosConfig dataclass
├── scanners/
│   ├── minimal.py       # MinimalScanner (zero-dep, 23 rules across 6 families)
│   ├── llm_guard.py     # LlmGuardScanner (extras: llm-guard)
│   ├── llama_firewall.py # LlamaFirewallScanner (extras: llamafirewall)
│   └── presidio.py      # PresidioScanner + anonymization (extras: presidio)
├── session/
│   ├── frequency.py     # FrequencyTracker (exp decay + rolling window)
│   ├── escalation.py    # 3-tier escalation (Tier 3 cannot be disabled)
│   ├── profiles.py      # 5 built-in + custom profiles
│   ├── guard.py         # ToolCallGuard
│   ├── audit.py         # AuditEmitter (verbosity-gated)
│   ├── alerting.py      # AlertManager (5 rules + rate limiting)
│   └── license.py       # JWT validation (local, parked for future use)
```

## Key Design Invariants

- **Pipeline never throws** — all errors caught and returned in `PipelineResult`.
- **Fail-mode defaults to `degraded`** — partial or total ML scanner failure blocks content; syntactic pre-filter (zero deps) always runs. Configurable to `open` or `closed`.
- **Zero required ML deps at base install** — scanner backends are pip extras, not hard deps. `pip install petasos` is lightweight; `pip install petasos[all]` is ~300MB.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vigil-Harbor/Petasos](https://github.com/Vigil-Harbor/Petasos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
