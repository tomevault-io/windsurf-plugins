---
trigger: always_on
description: This document describes how human contributors and AI assistants should operate inside this repository. Follow it whenever crafting code, tests, docs, or analyses so that new work is aligned with the architecture, workflows, and security expectations of Threat Thinker.
---

# Threat Thinker Agent Guide

This document describes how human contributors and AI assistants should operate inside this repository. Follow it whenever crafting code, tests, docs, or analyses so that new work is aligned with the architecture, workflows, and security expectations of Threat Thinker.

## 1. Mission & Scope
- Automate threat modeling from system diagrams (Mermaid, draw.io, raster images) and produce concise reports referencing OWASP ASVS/CWE items.
- Provide CLI, Web UI, and API entry points, plus a local RAG layer for augmenting LLM reasoning with curated documents.
- Support Threat Dragon JSON round-trips and HTML/Markdown/JSON report surfaces for reviewers and automation.
- Favor small, reviewable pull requests that include documentation and automated tests relevant to the change.

## 2. Repository Map & Architecture
- `src/threat_thinker/`
  - `main.py`: CLI parser/subcommands (`think`, `diff`, `kb`, `webui`, `serve`, `worker`), orchestrates parsers → hint processor → LLM adapters → analyzers → exporters.
  - `cliui.py`: lightweight UI helpers and verbose logging toggles used by CLI and tests.
  - `webui.py`: Gradio-based UI wiring.
  - `parsers/`: diagram ingestion (`mermaid_parser.py`, `drawio_parser.py`, `threat_dragon_parser.py`, `image_parser.py`) plus helpers — keep deterministic and side-effect free.
  - `hint_processor.py`: merges user hints with LLM-generated attributes.
  - `llm/`: provider-specific adapters, prompt builders, throttling/retry logic (`providers/`, `client.py`, `inference.py`).
  - `threat_analyzer.py`: deduplicates/denoises threats, filters via thresholds, ensures ASVS references as requested.
  - `exporters.py`: Markdown/JSON/HTML exporters and diff helpers (`diff_reports`, `export_diff_md`), including Threat Dragon JSON round-trip support.
  - `rag/`: local KB management (`local.py`) for chunking, embedding, semantic retrieval.
  - `serve/`: FastAPI service entry (`api.py`) and infrastructure (`auth.py`, `ratelimit.py`, `jobstore.py`, `schemas.py`, `config.py`).
  - `worker/`: background analysis runner for the `serve` job queue.
  - `service/`: service-layer helpers that connect analyzers and adapters.
  - `models.py`, `constants.py`, `zone_utils.py`: shared dataclasses, enums, and zone utilities (keep type hints updated when contracts change).
- `tests/`: mirrors `src/` package structure with pytest suites, fixtures, and golden files; includes `tests/serve/` for API/worker behaviors.
- `docs/`: architecture notes, tutorials, and specs (keep rationale here, not operational findings).
- `examples/`: canonical diagrams (`.mmd`, `.xml`, `.png`) and sample reports under `examples/**/reports/`.
- `reports/`: generated analysis artifacts in Markdown/JSON/HTML; never hand-edit these.
- `benchmarking/`: notebooks/scripts for perf or accuracy comparisons.
- `examples/demo-app/`: sample service configuration and Docker Compose setup.
- Root configuration: `pyproject.toml`, `uv.lock`, `pytest.ini` (Ruff config lives in `pyproject.toml`).
- `dist/`: built artifacts; do not edit by hand.

## 3. Execution Modes & Entry Points
- **CLI**: `think`, `diff`, `kb` for local usage and automation.
- **Web UI**: `webui` launches Gradio (default `http://localhost:7860`).
- **API service**: `serve` runs a FastAPI server configured via YAML (`examples/demo-app/serve.example.yaml`).
- **Worker**: `worker` processes queued analysis jobs for the API service.

## 4. Development Workflow
1. **Environment**
   ```bash
   uv venv                         # create .venv if absent
   source .venv/bin/activate
   uv sync --extra dev
   uv sync --extra dev --frozen    # reproduce the reviewed dependency set
   ```
   Use `pip install -e .[dev]` as a fallback if `uv` is unavailable.
2. **Credentials**: export one or more of `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or configure AWS credentials (`aws configure --profile ...` or `AWS_ACCESS_KEY_ID/...`) before running CLI/web UI/API features that contact LLMs.
3. **Iterate**
   - Run `ruff check src tests` and `ruff format` before committing.
   - Execute targeted pytest nodes during development (`pytest tests/parsers/test_mermaid_parser.py -k trust_boundary`) but always run `pytest` at the end.
   - When editing CLI flows, add fixtures to `tests/fixtures/` (e.g., diagrams, YAML hints) so regressions are reproducible offline.
4. **Artifacts**
   - CLI run:
     ```bash
     threat-thinker think \
         --mermaid examples/web/system.mmd \
         --infer-hints --topn 5 \
         --llm-api openai --llm-model gpt-4.1 \
         --format both \
         --out-md reports/web-report.md \
         --out-json reports/web-report.json
     ```
   - Diffing two reports:
     ```bash
     threat-thinker diff \
         --after reports/new-report.json \
         --before reports/old-report.json \
         --llm-api openai --llm-model gpt-4.1 \
         --out-md reports/diff.md --out-json reports/diff.json
     ```
   - RAG KB maintenance:
     ```bash
     threat-thinker kb build <kb_name> \
         --embedder openai:text-embedding-3-small \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melonattacker/threat-thinker](https://github.com/melonattacker/threat-thinker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
