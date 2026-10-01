---
trigger: always_on
description: This file states what holds in this repository, for an agent who would not
---

## About this file

This file states what holds in this repository, for an agent who would not
meet it at the point of need: couplings that span files, procedures that gate
a change, traps that fail silently or at a cost, and conventions that nothing
enforces. It is not a record of changes: what a change did belongs to its
commit and pull request. What an agent meets where it matters stays there, in
a module's docstring, an option's description or the user documentation. Add
a line only when an agent without it would go wrong, and write it as a fact
about the repository as it stands.

## The project

PyBADS is the Python port of the MATLAB BADS toolbox (Bayesian Adaptive
Direct Search, `acerbilab/bads`, the reference implementation): optimization
of black-box, possibly noisy, mildly expensive objectives with up to about
20 parameters, under bound and optional non-box constraints. Plain
NumPy/SciPy. The GP layer is the lab's `gpyreg` (`acerbilab/gpyreg`), a
sibling repository.

PyBADS does not import PyVBMC. It carries its own copies of classes that
PyVBMC also has (`Options`, `FunctionLogger`, `IterationHistory`, `Timer`,
`stats/`), which have diverged from PyVBMC's: a fix in one repository does
not reach the other. `VariableTransformer` is PyBADS's own and is not
PyVBMC's `ParameterTransformer`, although its documentation page is
`parameter_transformer.rst`.

- `dev/` holds the developer notes, plans, results and tooling.
  `dev/README.md` says where each kind of record goes; `dev/TODO.md` lists
  the open work.
- `dev/results/2026-09-23-codebase-survey.md` records the observed failures
  and the candidate defects of the port, not yet verified against MATLAB
  BADS. Check it before treating an oddity in the numerical code as
  intended, and record a fix or a verdict in its entry.
- `pybads/bads/README.md` lists the open porting work.
- `docsrc/` is the Sphinx source. `docs/` is its gitignored build output;
  the published site lives on the `gh-pages` branch.

## Setup and commands

The development environment is a venv at `.venv` (gitignored), with gpyreg
installed from a sibling checkout:

```console
python -m venv .venv                                    # then activate it
git clone https://github.com/acerbilab/gpyreg ../gpyreg
pip install -e "../gpyreg[dev]"
pip install -e ".[dev]"
pip install pre-commit && pre-commit install
```

The version comes from git tags through setuptools_scm, which writes the
gitignored `pybads/_version.py`. `test_init_conf.py` reads the installed
package metadata, so the suite needs the editable install, not only the
source on `sys.path`.

```console
python -m pytest --reruns=5 -x -vv                      # what CI runs
python -m pytest pybads/testing/bads/test_bads_optimization.py::test_sphere_opt
```

The tests live in `pybads/testing/`, mirroring the package, and default
discovery is limited to them (`testpaths` in `pyproject.toml`); the checks
under `dev/scripts/` run only when named by path.

The test job is defined once, in `.github/workflows/test-matrix.yml`, and
installs gpyreg at the commit pinned as `GPYREG_PIN` there: the tagged
commit of the release that `pyproject.toml` names as the minimum (CI reads
gpyreg's version from its tags, and an untagged commit reads lower, so pip
would install gpyreg from PyPI over the pinned checkout). A change that
needs a newer gpyreg moves both. `merge-tests.yml` runs the full matrix
(Ubuntu, Windows, macOS × Python 3.10–3.12) on a PR to `main` only when it
touches `pybads/`, `pyproject.toml` or `setup.py`; a PR that changes
anything else, the workflows included, runs no tests. `tests.yml` runs the
full matrix on dispatch and on the 13th and 28th of each month, the
scheduled run against gpyreg's `main` instead of the pin (the drift
detector), and a smoke run (Ubuntu, Python 3.12) on each push to a `dev*`
branch that touches the package. `docs.yml` rebuilds the docs on every push
to `main` and commits them to `gh-pages`.

A release is a tag `vX.Y.Z` on `main` and a GitHub release published from
it: `release.yml` builds the package with `build.yml` and uploads it to
PyPI by trusted publishing, through the `pypi` environment, which admits
only `v*` tags. No token is stored.

Formatting is enforced by the pre-commit hooks alone (black at line length
79 on every Python file and the notebooks' code cells, isort with the black
profile, pycln); no CI job checks it, and the whole tree passes them. The
commit that first formatted the tree is listed in `.git-blame-ignore-revs`;
`git config blame.ignoreRevsFile .git-blame-ignore-revs` hides it from
`git blame`.

`pyproject.toml` is authoritative; `setup.py` is a shim. It names only
`pybads` and `pybads.examples` as packages: the subpackages and the `.ini`
option files reach the wheel through `include-package-data` and
setuptools_scm's file finder, which takes only files tracked by git, so a
new module or data file ships only once committed. The tests ship in the
wheel: the conda-forge recipe runs them from the installed package
(`pytest --pyargs pybads`). What they need is the `test` extra, which CI
installs and `dev` includes; no package module imports pytest.

Docstrings are numpydoc, with `OptimizeResult`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acerbilab/pybads](https://github.com/acerbilab/pybads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
