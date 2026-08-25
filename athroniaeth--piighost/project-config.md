---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PIIGhost is a composable PII de-identification pipeline for LLM agents. It detects, anonymizes, and deanonymizes sensitive entities using pluggable detectors (regex, GLiNER2, spaCy, Transformers, LLM), with a LangChain middleware for LangGraph agents, TOML/JSON-driven configuration, and an HTTP client for the companion `piighost-api` server. The design is hexagonal: every stage is a port (an `Any*` runtime_checkable Protocol) with a `Base*` template ABC, and configuration couples to the core in one direction only.

## Development Commands

```bash
uv sync                      # Install dependencies (dev group included)
make format                  # Auto-fix: ruff format + ruff check --fix
make lint                    # Blocking gate: ruff format --check + ruff check + pyrefly + bandit
uv run pytest                # Run all tests (integration tests deselected by default)
uv run pytest tests/pipeline/test_pipeline.py -k "test_name"  # Run a single test
uv run pytest -m integration # Run integration tests (load torch/gliner2/spacy)
make docs-build              # Build EN + FR docs (zensical)
make docs-watch              # Serve EN docs with live reload (docs-watch-fr for FR)
```

`make lint` is a real gate (non-mutating, fails on any issue); use `make format` to auto-fix. Tests marked `integration` load heavy optional dependencies and are excluded by the default `addopts`. `asyncio_mode = "auto"`, so async tests need no decorator.

## Architecture

Each pipeline stage is a package under `src/piighost/components/` whose `base.py` holds the port (`Any*` Protocol) and a `Base*` template ABC; concrete adapters are sibling modules.

### Anonymization Pipeline

`AnonymizationPipeline` (`pipeline/base.py`, extends `BaseAnonymizationPipeline`) runs the stages in order. Only the detector is required: the linker defaults to `ExactEntityLinker` and the anonymizer to `Anonymizer(LabelCounterPlaceholderFactory())`, and the overlap, expand, entity-resolve, override, and guard stages default to disabled.

1. **Detect**: `AnyDetector` (`components/detector/base.py`). `RegexDetector` (prebuilt pattern catalogs in `components/detector/patterns/`: generic, us, eu, fr; no checksum validators, so it matches on shape alone and never drops an OCR-mangled value), `ExactMatchDetector` (tests), `CompositeDetector` (runs several concurrently), `ChunkedDetector` (overlapping-chunk splitting for long text), and the model detectors `Gliner2Detector` / `SpacyDetector` / `TransformersDetector` (extend `BaseNERDetector` for label mapping) plus `LLMDetector` (LangChain structured output).
2. **Override** (optional): `AnyDetectionOverride` (`components/override/`) imposes a server whitelist and blacklist on every detection set.
3. **Resolve overlaps** (optional): `AnyOverlapResolver` / `ConfidenceOverlapResolver` keeps the highest-confidence detection when spans overlap.
4. **Expand** (optional): `AnyDetectionExpander` / `WordBoundaryExpander` adds missed occurrences.
5. **Link**: `AnyEntityLinker` / `ExactEntityLinker` groups detections that share a value and label into an `Entity`.
6. **Resolve entities** (optional): `AnyEntityResolver` / `MergeEntityResolver` (union-find) or `FuzzyEntityResolver` (Jaro-Winkler, `fuzzy` extra).
7. **Anonymize**: `AnyAnonymizer` / `Anonymizer` applies span-based replacement using an `AnyPlaceholderFactory`.
8. **Guard rail** (optional): `AnyGuardRail` (`components/guard/`) re-checks the output for residual PII and raises `PIIRemainingError`. `DetectorGuardRail` re-runs a detector, `LLMGuardRail` (`guard/llm.py`) prompts an LLM to ignore placeholders, `ModerationGuardRail` (`guard/moderation.py`, `mistral` extra) backs the check with Mistral.

Data models (`Entity`, `Detection`, `Span`) are frozen dataclasses under `models/`. Tests use `ExactMatchDetector` to avoid loading real models.

### Placeholder Factories & Preservation Tags

`components/placeholder/tags.py` defines a phantom-type hierarchy (a `str` subclass) describing what a token preserves: label (`<PERSON>` vs `[REDACT]`), identity (`<<PERSON:1>>` uniquely identifies), realism (Opaque / Hashed), and shape (masks like `j***@mail.com`). Pipelines are generic on this tag. Factories in `components/placeholder/`: `RedactPlaceholderFactory`, `LabelPlaceholderFactory`, `MaskPlaceholderFactory`, `LabelCounterPlaceholderFactory` (`<<PERSON:1>>`), `LabelHashPlaceholderFactory` (pepper via `PIIGHOST_HASH_PEPPER`). The middleware requires a token that preserves recognizable identity (`PreservesRecognizableIdentity`) so it can find and restore it. There is no Faker factory.

### Conversation Layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Athroniaeth/piighost](https://github.com/Athroniaeth/piighost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
