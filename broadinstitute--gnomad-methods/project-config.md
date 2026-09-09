---
trigger: always_on
description: Shared Hail utility library for gnomAD pipelines, published to PyPI as the
---

# gnomad_methods Project Reference

## Project Overview

Shared Hail utility library for gnomAD pipelines, published to PyPI as the
`gnomad` package. Provides reusable functions for variant QC, sample QC,
constraint analysis, Ensembl VEP processing, resource management, and general
genomics operations. Used as a dependency by `gnomad_qc`, `gnomad-constraint`,
and other gnomAD repos — treat the public API as something downstream code
depends on. See the README for the list of sibling repos.

**This is a library, not a pipeline.** It exposes APIs that other repos import;
it has no `__main__` entry point and is not run via Dataproc directly.
Public-API changes ripple through every consuming repo, so prefer additive
changes (new functions, new optional parameters with safe defaults) over
breaking renames.

**This is a public repo of genomic utility functions**, and the point of it is
reusability: future gnomAD team members and external users need to find and
understand code they can reuse. External groups have repeatedly asked the gnomAD
team for functionality that already existed here but that they couldn't find.
Optimize new code for discoverability — clear names, tight scopes, accurate
docstrings — not for cleverness.

## Repo Layout

**The repo layout lives in the README**, along with the list of related gnomAD
repos — read it there rather than duplicating it here. If a change adds,
removes, or moves a subpackage, update the layout table in the README as part of
the same pull request.

### What belongs here vs. in gnomad_qc

Generalized, reusable functions belong in gnomad_methods; gnomAD release
pipeline code belongs in `gnomad_qc`. Thin wrapper functions are acceptable
here.

The dependency runs one way only: `gnomad_qc` imports gnomad_methods, and
gnomad_methods never imports `gnomad_qc`. The boundary is still confusing in
practice, though, and newcomers routinely search both repos to find one thing —
partly because a `gnomad_qc` function is often a thin wrapper around a
gnomad_methods function of a similar name. Trace the call chain before assuming
which repo owns a behavior.

## Finding Code

**Do not guess — read and confirm.** Do not assume a function exists, guess its
module from memory, or reason about its performance from its name. Function
names and locations change between versions.

This is the most common way work here goes wrong. A concrete example: asked to
make a gnomad_methods function more efficient, Claude proposed several changes
and asserted they would speed things up; none did, because it had not read the
function body or how the caller was using it. Read the actual implementation and
the actual call site before proposing a change, and don't claim a performance
win you haven't measured.

- **Search before writing**: before adding a new utility, grep the package for
  related keywords (`grep -rn "allele_frequency" gnomad/`). This library is
  large and the function you need often already exists, possibly under a
  different name than you'd guess.
- **Module docstrings and section headers**: each module starts with a
  docstring describing its scope — read it to confirm you're in the right
  place before diving into functions.
- **Tests show intended usage**: `tests/` mirrors the package layout; a test
  file is often the best usage example for a function.
- **Downstream usage**: the sibling repos listed in the README (most usefully
  `gnomad_qc`, often checked out at `../gnomad_qc`) show how functions are used
  in real pipelines. Check there before changing a signature.
- **Generated docs**: https://broadinstitute.github.io/gnomad_methods/ — built
  from docstrings, so the docstring in the source is always authoritative.

## Development Commands

```bash
pip install -r requirements.txt -r requirements-dev.txt   # setup
python3 -m pre_commit install                             # install hooks

black gnomad tests                  # format
isort --profile black --filter-files gnomad tests
autopep8 --in-place gnomad          # comment formatting
pydocstyle gnomad tests             # docstring check
./lint                              # pylint (gnomad + tests)
python -m pytest                    # all tests
python -m pytest tests/utils/test_vep.py  # one file
./docs/build.sh                     # build docs (regenerates API reference)
```

Formatting config lives in `pyproject.toml`, `.pydocstylerc`, `.pylintrc`, and
`.pre-commit-config.yaml`. Black runs in preview mode with the default
88-character line length.

### Dependencies

Adding a new third-party library is a change to the package's install
requirements, and every consuming repo inherits it. Before introducing one,
check that it is compatible with the versions already pinned in
`requirements.txt` (and `requirements-dev.txt` for test-only libraries) — the
Hail, pandas, and numpy pins are the ones that usually conflict. If a new
library needs a pin loosened or bumped, **make that dependency update part of
the same pull request**, and say so in the PR description so reviewers know the
install surface changed.

## Pull Requests

**Run the `/review` skill on every pull request before requesting review from a
team member** (Ben Weisburd's skill:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [broadinstitute/gnomad_methods](https://github.com/broadinstitute/gnomad_methods) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
