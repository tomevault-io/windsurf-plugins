---
trigger: always_on
description: AGENTS.md — Machine-focused operational guide for the sdialog repository.
---

<!--
AGENTS.md — Machine-focused operational guide for the sdialog repository.
This file is intentionally concise, action‑oriented, and complementary to README.md.
Agents: Prefer following explicit commands / conventions here over guessing.
Human contributors: See README.md + CONTRIBUTING.md for narrative context.
-->

# AGENTS Guidelines for This Repository

Synthetic Dialogue Generation, Orchestration, Evaluation, Interpretability.

Focus for agents:
1. Reproducible environment & dependency handling
2. Safe model / API usage & configuration overrides
3. Standard commands (build, test, lint, docs, packaging)
4. Dataset & artifacts locations
5. Extension points (personas, orchestrators, evaluators, inspectors)
6. Performance & caching knobs
7. Contribution / PR hygiene
8. Security / privacy considerations

If an instruction here conflicts with user chat input, defer to the user. For file‑local changes prefer editing minimal regions.

---

## 1. Repository Layout (key paths only)

```
sdialog/                Core library (packaged via pyproject.toml)
  requirements.txt      Runtime + dev dependencies (dynamic in pyproject)
  src/ or package root  (Package modules live directly under sdialog/)
  tutorials/            Example notebooks & advanced usage
  docs/                 Sphinx docs (ReadTheDocs)
  AGENTS.md             (This file)
  README.md             Human overview
  CONTRIBUTING.md       Contribution guidelines
  LICENSE               MIT
datasets/ | Datasets/   External dialogue / STAR dataset snapshots (read-only)
AutoTOD/, AgenTOD/, task-oriented-dialogue/  Related research / comparative tooling (not installed by default)
JSALT/                  Workshop materials & experiments
```

Only the `sdialog` Python package is published to PyPI. Other folders are experimental/supporting and may have separate requirements.

---

## 2. Environment Setup

Supported Python: >=3.9 (see `pyproject.toml`). Recommended fresh virtual environment.

### Quick install (library usage)
```
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -e .[dev]  # if an extras block is later added; otherwise:
pip install -r sdialog/requirements.txt
pip install -e sdialog
```

### Clean reinstall
```
pip uninstall -y sdialog || true
pip install -e sdialog
```

### Dependency resolution notes
* `pyproject.toml` uses `setuptools.dynamic` to read `requirements.txt`.
* Pin additions: modify `sdialog/requirements.txt` then reference in PR.
* Avoid silently upgrading core ML libs (`torch`, `transformers`) without noting compatibility.
* If adding optional backend (OpenAI / Ollama / AWS / Google), ensure minimal import‑time cost; guard imports.

### GPU / Torch
* Do not auto‑install CUDA wheels; leave to user environment.
* If a test requires GPU, skip gracefully when `torch.cuda.is_available()` is False.

---

## 3. LLM Configuration & Global State

Central API:
```
import sdialog
sdialog.config.llm("provider:model", temperature=0.7, top_p=0.9)
sdialog.config.llm_params(max_tokens=512)
```
Providers use prefix naming:
* `openai:MODEL`
* `huggingface:REPO`
* `ollama:MODEL`
* `amazon:bedrock-model-id`
* `google:genai-model-id`

When modifying code that instantiates models:
* Always allow explicit `model=` override in class constructors.
* Keep default fallback: `config["llm"]["model"]`.
* Avoid hard‑coding API keys; rely on environment variables (e.g., `OPENAI_API_KEY`, `GOOGLE_API_KEY`). Never commit keys.

Think segments: If `think=True`, internal prompts may contain `<think>...</think>` sections; pattern customizable via `thinking_pattern`.

Tools: Agents accept simple Python callables; treat them as pure (side‑effect‑light) unless clearly documented.

---

## 4. Core Command Cheat‑Sheet

| Task | Command |
|------|---------|
| Install dev deps | `pip install -r sdialog/requirements.txt` |
| Editable install | `pip install -e sdialog` |
| Lint (flake8) | `flake8 sdialog` |
| Run tests (pytest) | `pytest -q` |
| Coverage | `pytest --cov=sdialog --cov-report=term-missing` |
| Build docs | `pip install -r sdialog/docs/requirements.txt && (cd sdialog && make -C docs html || sphinx-build -b html docs docs/_build/html)` |
| Package sdist/wheel | `python -m build` (if build backend tooling added) |
| Update version | Edit `sdialog/util/__init__.py` (where `__version__` lives) |
| Format tables | Use `sdialog.util.dict_to_table(..., markdown=True)` |

Tests assume importable `sdialog`; ensure editable install before running.

---

## 5. Data & Artifacts

STAR dataset utilities live under `sdialog/datasets`. External raw STAR data likely mirrored under `Datasets/STAR` or `datasets/STAR`.

Agent operations MUST NOT mutate dataset source files. For synthetic generation:
* Write outputs (dialogs JSON) into a new folder (e.g., `outputs/` or `results/`).
* Use `Dialog.to_file()` for serialization; prefer `.json` extension.

Large artifacts (embeddings, cached evaluations) should be placed in a git‑ignored path (e.g., `cache/`, configurable via `sdialog.config.set_cache(path, enable=True)`).

---

## 6. Personas, Agents, Orchestrators (Extension Points)

Subclassing patterns:
* Personas: Inherit from `BaseAttributeModel` (see `sdialog.personas`). Keep field names snake_case.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [idiap/sdialog](https://github.com/idiap/sdialog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
