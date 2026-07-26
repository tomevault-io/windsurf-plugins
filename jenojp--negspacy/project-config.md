---
trigger: always_on
description: Guidance for Claude Code when working on this repository. Read this in full before making any changes.
---

# CLAUDE.md

Guidance for Claude Code when working on this repository. Read this in full before making any changes.

## Project purpose

`negspacy` is a spaCy v3 pipeline component that identifies negated entities and spans in text, based on the NegEx algorithm (Chapman et al., 2001). It is published to PyPI as `negspacy` and is widely used in clinical NLP and biomedical research.

The package is small, pure-Python, and has two responsibilities:
1. Provide a `negex` pipeline factory that sets a boolean `._.negex` extension on entities or spans.
2. Provide a `termset` API for built-in and user-defined negation pattern lists.

## Public API — DO NOT break without an explicit major version bump

Downstream users and research papers depend on these surfaces. Breaking any of them is a breaking change and requires coordination with the maintainer, not a silent PR.

### Factory registration
The component must remain registered as `"negex"` via `@Language.factory("negex", ...)` so that `nlp.add_pipe("negex", config={...})` continues to work.

### Default config keys
These keys, names, and default semantics must be preserved:
- `neg_termset` — dict with keys `pseudo_negations`, `preceding_negations`, `following_negations`, `termination`. Defaults to `termset("en_clinical").get_patterns()`.
- `ent_types` — optional list of entity label strings to filter on. Default `None` (all ents).
- `extension_name` — name of the boolean attribute set on ents/spans. Default `"negex"`, accessed as `ent._.negex`.
- `chunk_prefix` — optional list of strings that may appear as a chunk prefix (e.g., `["no"]`). Default `None`.
- `span_keys` — optional list of `doc.spans` keys to process instead of (or in addition to) `doc.ents`. Default `None`.

### Extension attribute
`Token`, `Span`, and/or entity objects expose a boolean attribute registered under `extension_name` (default `negex`). `ent._.negex` and `span._.negex` are the documented access patterns. Do not rename, re-type, or change the default without a major version bump.

### Termset API
`negspacy.termsets.termset(language)` must continue to accept the three built-in strings:
- `"en"` — general English
- `"en_clinical"` — default; adds clinical phrases
- `"en_clinical_sensitive"` — adds historical/irrelevance phrases

The returned object must continue to expose `get_patterns()`, `add_patterns(dict)`, and `remove_patterns(dict)` with the existing dict schema.

### Pattern category names
The four pattern keys are a public contract: `pseudo_negations`, `preceding_negations`, `following_negations`, `termination`. Note the historical misspelling `psuedo_negations` existed in pre-1.0 versions — the correct spelling `pseudo_negations` is the current one and must be preserved.

## Target repository layout (post-modernization)

The modernization effort moves the project to a standard `src/` layout with a dedicated `tests/` directory and a single `pyproject.toml` as the source of truth.

```
negspacy/                       # repo root
  src/
    negspacy/
      __init__.py
      negation.py               # Negex class + @Language.factory("negex")
      termsets.py               # termset() + built-in en / en_clinical / en_clinical_sensitive
      py.typed                  # PEP 561 marker (type hints are shipped)
  tests/
    __init__.py
    conftest.py                 # shared fixtures (spaCy model loading, etc.)
    test_negation.py            # split out from old negspacy/test.py
    test_termsets.py            # split out from old negspacy/test.py
  .github/
    workflows/
      ci.yml                    # replaces azure-pipelines.yml
      release.yml               # tag-triggered PyPI publish via OIDC
    ISSUE_TEMPLATE/
  .pre-commit-config.yaml       # ruff + standard hygiene hooks
  pyproject.toml                # single source of truth (PEP 621)
  CLAUDE.md                     # this file
  CHANGELOG.md                  # Keep a Changelog format, seeded from git history
  CONTRIBUTING.md
  README.md
  LICENSE                       # MIT
```

Files to be removed during modernization: `setup.py`, `setup.cfg`, `requirements.txt`, `azure-pipelines.yml`, and the legacy `negspacy/test.py` (contents moved into `tests/`).

Adopting the `src/` layout is deliberate: it prevents accidental imports of the in-development package during testing and forces tests to run against the installed package, which catches packaging bugs early.

## Packaging and dependencies

`pyproject.toml` is the single source of truth. Use **hatchling** as the build backend (simpler than setuptools, zero-config, official PyPA).

Runtime dependencies live in `[project.dependencies]`. Development dependencies live in `[project.optional-dependencies]` under a `dev` extra, which installs lint, test, and docs tooling together. This is the convention most Python users expect and works with `pip install -e ".[dev]"` out of the box.

Sketch:
```toml
[project]
name = "negspacy"
dynamic = ["version"]
requires-python = ">=3.9"
dependencies = [
    "spacy>=3.8,<4.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8",
    "pytest-cov",
    "ruff",
    "pre-commit",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.version]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jenojp/negspacy](https://github.com/jenojp/negspacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
