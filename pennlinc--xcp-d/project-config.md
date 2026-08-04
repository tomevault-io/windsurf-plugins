---
trigger: always_on
description: This file provides instructions for AI coding agents and human maintainers working on **XCP-D**, a BIDS App for robust postprocessing of fMRI data.
---

# AGENTS.md -- XCP-D

This file provides instructions for AI coding agents and human maintainers working on **XCP-D**, a BIDS App for robust postprocessing of fMRI data.

---

## Shared Instructions (All PennLINC BIDS Apps)

The following conventions apply equally to **qsiprep**, **qsirecon**, **xcp_d**, and **aslprep**. All four are PennLINC BIDS Apps built on the NiPreps stack.

### Ecosystem Context

- These projects belong to the [NiPreps](https://www.nipreps.org/) ecosystem and follow its community guidelines.
- Core dependencies include **nipype** (workflow engine), **niworkflows** (reusable workflow components), **nireports** (visual reports), **pybids** (BIDS dataset querying), and **nibabel** (neuroimaging I/O).
- All four apps are containerized via Docker and distributed on Docker Hub under the `pennlinc/` namespace.
- Contributions follow the [NiPreps contributing guidelines](https://www.nipreps.org/community/CONTRIBUTING/).

### Architecture Overview

Every PennLINC BIDS App follows this execution flow:

```
CLI (parser.py / run.py)
  -> config singleton (config.py, serialized as ToML)
    -> workflow graph construction (workflows/*.py)
      -> Nipype interfaces (interfaces/*.py)
        -> BIDS-compliant derivative outputs
```

- **CLI** (`<pkg>/cli/`): `parser.py` defines argparse arguments; `run.py` is the entry point; `workflow.py` builds the execution graph; `version.py` handles `--version`.
- **Config** (`<pkg>/config.py`): A singleton module with class-based sections (`environment`, `execution`, `workflow`, `nipype`, `seeds`). Config is serialized to ToML and passed across processes via the filesystem. Access settings as `config.section.setting`.
- **Workflows** (`<pkg>/workflows/`): Built using `nipype.pipeline.engine` (`pe.Workflow`, `pe.Node`, `pe.MapNode`). Use `LiterateWorkflow` from `niworkflows.engine.workflows` for auto-documentation. Every workflow factory function must be named `init_<descriptive_name>_wf`.
- **Interfaces** (`<pkg>/interfaces/`): Custom Nipype interfaces wrapping external tools or Python logic. Follow standard Nipype patterns: define `_InputSpec` / `_OutputSpec` with `BaseInterfaceInputSpec` / `TraitedSpec`, implement `_run_interface()`.
- **Utilities** (`<pkg>/utils/`): Shared helper functions. BIDS-specific helpers live in `utils/bids.py`.
- **Reports** (`<pkg>/reports/`): HTML report generation using nireports.
- **Data** (`<pkg>/data/`): Static package data (config files, templates, atlases). Accessed via `importlib.resources` or the `acres` package.
- **Tests** (`<pkg>/tests/`): Pytest-based. Unit tests run without external data. Integration tests are gated behind pytest markers and are skipped by default.

### Workflow Authoring Rules

1. Every workflow factory function must be named `init_<name>_wf` and return a `Workflow` object.
2. Use `LiterateWorkflow` (from `niworkflows.engine.workflows`) to enable automatic workflow graph documentation.
3. Define `inputnode` and `outputnode` as `niu.IdentityInterface` nodes to declare the workflow's external API.
4. Connect nodes using `workflow.connect([(source, dest, [('out_field', 'in_field')])])` syntax.
5. Add `# fmt:skip` after multi-line `workflow.connect()` calls to prevent ruff from reformatting them.
6. Include a docstring with `Workflow Graph` and `.. workflow::` Sphinx directive for auto-generated documentation.
7. Use `config` module values (not function parameters) for global settings inside workflow builders.

### Interface Conventions

1. Input/output specs use Nipype traits (`File`, `traits.Bool`, `traits.Int`, etc.).
2. `mandatory = True` for required inputs; provide `desc=` for all traits.
3. Implement `_run_interface(self, runtime)` -- never `run()`.
4. Return `runtime` from `_run_interface`.
5. Set outputs via `self._results['field'] = value`.

### Config Module Usage

```python
from <pkg> import config

# Read a setting
work_dir = config.execution.work_dir

# Serialize to disk
config.to_filename(path)

# Load from disk (in a subprocess)
config.load(path)
```

The config module is the single source of truth for runtime parameters. Never pass global settings as function arguments when they are available via config.

### Testing Conventions

- **Unit tests**: Files named `test_*.py` in `<pkg>/tests/`. Must not require external neuroimaging data or network access.
- **Integration tests**: Decorated with `@pytest.mark.<marker_name>`. Excluded by default via `addopts` in `pyproject.toml`. Require Docker or pre-downloaded test datasets.
- **Fixtures**: Defined in `conftest.py`. Common fixtures include `data_dir`, `working_dir`, `output_dir`, and `datasets`.
- **Coverage**: Configured in `pyproject.toml` under `[tool.coverage.run]` and `[tool.coverage.report]`.

### Documentation

- Built with Sphinx using `sphinx_rtd_theme`.
- Source files in `docs/`.
- Workflow graphs are auto-rendered via `.. workflow::` directives that call `init_*_wf` functions.
- API docs generated via `sphinxcontrib-apidoc`.
- Bibliography managed with `sphinxcontrib-bibtex` and `boilerplate.bib`.

### Docker


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PennLINC/xcp_d](https://github.com/PennLINC/xcp_d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
