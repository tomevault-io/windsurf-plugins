---
trigger: always_on
description: pylxd is the Python client library for the [LXD](https://ubuntu.com/lxd) REST API.
---

# AGENTS.md — pylxd Agent Instructions

pylxd is the Python client library for the [LXD](https://ubuntu.com/lxd) REST API.
Package: `pylxd` on PyPI. License: Apache-2.0. Default branch and PR target: `main`.

These instructions are for any AI coding agent, whichever tool or model drives it.
The human contribution guide is `doc/source/contributing.rst`; read it too.

## Prerequisites

- Python 3.10 or higher and [tox](https://tox.wiki/) 4.21 or higher.
- Every check runs through tox. Environments are defined in `pyproject.toml` under
  `[tool.tox]`; there is no `tox.ini`, `Makefile`, or `requirements.txt`.
- Linting, type checking, and unit tests need no LXD daemon.

```bash
python3 -m venv venv
. ./venv/bin/activate
pip install --upgrade pip tox
```

tox needs network access the first time it builds an environment. If you have none,
say which checks you skipped in the pull request; CI runs all of them.

## Repository layout

```
pylxd/
  __init__.py        Re-exports Client and EventType
  client.py          Client, _APINode (REST tree traversal), transports, events
  managers.py        BaseManager and one manager class per resource
  exceptions.py      Exception hierarchy (LXDAPIException, NotFound, ...)
  models/
    _model.py        Model base class, Attribute descriptors, save/delete/sync
    <resource>.py    One module per LXD resource
    tests/           Unit tests for the models
  tests/
    mock_lxd.py      Mock LXD responses (RULES) shared by every unit test
    testing.py       PyLXDTestCase base class and test helpers
    test_client.py   Unit tests for the client
integration/         Integration tests (need a real LXD daemon)
doc/source/          Sphinx documentation, one page per resource plus api.rst
contrib_testing/     Manual scripts, not run by CI
```

No tracked file is generated. `build/`, `dist/`, `*.egg-info`, `.tox/`, and `doc/build/`
are local artifacts; never commit them.

## Build

pylxd is pure Python and has no build step for development. `tox -e release` builds the
sdist and wheel; maintainers use it when cutting a release.

## Architecture

1. `pylxd.Client` is the primary public entry point. `client.api` is an `_APINode`
   that maps attribute and item access onto REST paths:
   `client.api.instances["c1"].get()` issues `GET /1.0/instances/c1`.
2. Managers in `pylxd/managers.py` expose each model's class methods (`get`, `all`,
   `create`, `exists`) on the client, for example `client.instances.get("c1")`.
3. Models in `pylxd/models/` declare their fields with `model.Attribute(...)` and
   inherit `sync`/`save`/`delete` from `Model`.

Adding or extending a resource usually touches, in this order:

| Step | File |
|------|------|
| Model and its attributes | `pylxd/models/<resource>.py` |
| Export (new classes only) | `pylxd/models/__init__.py` |
| Manager wiring (new resources only) | `pylxd/managers.py`, `pylxd/client.py` |
| Mock responses | `pylxd/tests/mock_lxd.py` |
| Unit tests | `pylxd/models/tests/test_<resource>.py` |
| Integration tests | `integration/test_<resource>.py` |
| Documentation | `doc/source/<resource>.rst`, `doc/source/api.rst` |

## Validate before committing

Run these in order. Each must pass before moving to the next.

```bash
# 1. Auto-format (isort + Black)
tox -e format

# 2. Lint (Black and isort checks, flake8, check-manifest)
tox -e lint

# 3. Type check (mypy)
tox -e check

# 4. Unit tests (pytest with doctests)
tox -e unit

# 5. Coverage; the total must not drop
tox -e coverage

# 6. Only if doc/ changed
tox -e doc

# 7. Only if a shell script changed (CI runs this in the lint job)
shellcheck integration/run-integration-tests*
```

`tox -e lint` runs `check-manifest`. A new tracked file that is not Python must be
either shipped through `MANIFEST.in` or listed under `[tool.check-manifest]` in
`pyproject.toml`.

Run a subset of the unit tests by passing pytest arguments after `--`:

```bash
tox -e unit -- pylxd/models/tests/test_project.py -k test_get
```

## Tests

### Unit tests

- Subclass `pylxd.tests.testing.PyLXDTestCase`. It mocks HTTP with `requests-mock`,
  loads `mock_lxd.RULES`, and provides `self.client`.
- Never contact a real LXD daemon from a unit test.
- Add shared responses to `RULES` in `pylxd/tests/mock_lxd.py`. Override a response
  for a single test with `self.add_rule({...})`.
- The mock server advertises no API extensions. Enable them per test with
  `testing.add_api_extension_helper(self, ["extension_name"])`.
- Assert on what was sent with `self.last_matching_request(method, url)`.
- Mark lines that genuinely cannot be tested with `# pragma: no cover` and say why.

### Integration tests

- Subclass `integration.testing.IntegrationTestCase` and register cleanups with
  `self.addCleanup(...)` for everything the test creates.
- Do not run `integration/run-integration-tests` or `tox -e integration` on a
  machine whose LXD matters. They install packages with `sudo`, change the LXD
  server configuration (`core.https_address`, trust store, default storage pool),
  and create and delete real resources.
- The safe local option is `tox -e integration-in-lxd`. It needs a working local LXD
  that allows nesting, launches ephemeral containers, and runs the suite inside them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [canonical/pylxd](https://github.com/canonical/pylxd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
