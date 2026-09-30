---
trigger: always_on
description: The delta for this package. Everything repository-level — the workspace layout, the
---

# AGENTS.md — packages/sphinx-test-reports

The delta for this package. Everything repository-level — the workspace layout, the
commands, the lock, lint/format/type-check configuration, the release recipe, the pull
request requirements — is in the ROOT [`AGENTS.md`](../../AGENTS.md), and this file does not
repeat it. What is here is what an agent has to know that is true of sphinx-test-reports and
not of the workspace.

## Project Overview

sphinx-test-reports turns test results into needs. It has **three surfaces, and only one of
them is a Sphinx extension** — which is the single most important thing to know about this
package, because it shapes the manifest, the CI and the split that is coming:

- **the extension** — `test-file`, `test-suite`, `test-case`, `test-report`, `test-results`
  and `test-env` directives, which read JUnit / ctest / googletest XML and tox-envreport
  JSON and create sphinx-needs items from them, plus the `tr_link` dynamic function;
- **the converter** — a `test-reports` console script that turns the same reports into a
  `needs.json` **without running Sphinx at all**;
- **the pytest plugin** — `sphinxcontrib.test_reports.pytest_plugin`, which writes the XML
  shape the extension reads, including per-case properties for traceability.

So **Sphinx and sphinx-needs are an `[project.optional-dependencies]` extra, not
dependencies**: `pip install sphinx-test-reports` gets you `lxml` and the last two surfaces;
`pip install "sphinx-test-reports[sphinx]"` gets you the extension. The published wheel's
`Requires-Dist` is `lxml` alone. Two things in this repository exist because of that — the
`toolchain-free` CI job and this package's `compat-requirements.txt` — and both are
described below.

## Package structure

```text
pyproject.toml          # `[project]`, `[project.urls]`, `[project.scripts]` and
                        #   `[tool.flit.module]`. NOT ruff, ty, pytest or dependency
                        #   groups: those are the root's, and check (7) refuses them here
compat-requirements.txt # released deps the compat cell needs -- see "Releasing" below
.readthedocs.yaml       # this package's RTD project; its paths are REPOSITORY-root relative
AUTHORS · LICENSE · README.rst
design/                 # import-commit-map.txt: old hash -> new hash for the 2026-09 import

src/sphinxcontrib/test_reports/
├── __init__.py         # the lazy `setup` re-export; `sphinxcontrib` is a PEP 420 namespace
├── test_reports.py     # the extension entry point: directives, config values, fields
├── cli.py              # the `test-reports` converter command
├── pytest_plugin.py    # the pytest plugin
├── junitparser.py · jsonparser.py · results.py · identity.py · fields.py
│                       # the toolchain-free core: parsers, the result vocabulary, the
│                       #   deterministic case IDs, the one field table both writers share
├── projectconfig.py    # the `[test_reports]` ubproject.toml model and its discovery walk
├── needs_export.py · remote.py · config.py · environment.py · exceptions.py · toolchain.py
├── directives/         # one module per directive, all inheriting TestCommonDirective
├── functions/          # `tr_link`, a sphinx-needs dynamic function
├── css/ · schemas/JUnit.xsd
└── directives/test_report_template.txt   # the DEFAULT tr_report_template -- it SHIPS

tests/                  # `tests/__init__.py` is why this path is not in the root testpaths
docs/                   # conf.py sits IN the source dir; changelog.rst is stamped by `bump`
```

## The things that are true here and nowhere else

### The module name is DOTTED, and one workspace fence is silent because of it

This package installs into the `sphinxcontrib` PEP 420 namespace, so its import name is
`sphinxcontrib.test_reports` — not the distribution name with `-` → `_`. It says so in
`[tool.flit.module] name`, and three readers honour that key: `check_workspace.Member.module`,
`tools/src/sn_tools/import_check.py`, and (since this package's import) the `module=` step of
`.github/workflows/release.yaml`.

**`check_workspace.py` check (5) prints NO line at all for this member, and that is expected
today.** Every other member gets an `OK … __version__ == <version>` line; this one is absent,
and absence is not something a reader notices — so it is written down here. Two independent
reasons, either of which alone would be enough:

1. `module_version()` joins `member.module` as ONE path component, so it looks for
   `src/sphinxcontrib.test_reports/__init__.py` — a directory that cannot exist. A dotted
   name can never resolve.
2. **This package has no `__version__` literal anywhere.** `check_module_version` treats a
   module without one as "not an error" by design, so even a non-dotted name would print
   nothing until the literal exists. (`test_reports.py` carries a separate, hand-written
   `VERSION = "2.0.0"`, which no gate reads.)

What the gap costs is that `[project] version` and a module literal could drift apart —
which is nil in practice while nothing bumps this member. **Both reasons dissolve together**
when the package is renamed and gains a real `__version__`, which is the release that follows
this import. Until then, do not read check (5)'s silence as a pass.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [useblocks/sphinx-needs](https://github.com/useblocks/sphinx-needs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
