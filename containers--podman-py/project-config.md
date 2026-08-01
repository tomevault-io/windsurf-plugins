---
trigger: always_on
description: ![PODMAN logo](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png)
---

# AI Agent Guide for podman-py Development

![PODMAN logo](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png)

Unless noted otherwise, paths below are relative to the root of a [podman-py](https://github.com/containers/podman-py) source checkout. You can keep this file as `AGENTS.podman-py.md` in a parent workspace, or name it `AGENTS.md` at the root of a clone. When in doubt, treat [CONTRIBUTING.md](CONTRIBUTING.md) and the checked-in [Makefile](Makefile) / [tox.ini](tox.ini) / [pyproject.toml](pyproject.toml) as the authority.

## Persona

This guide is for AI coding assistants (for example Claude, ChatGPT, Copilot). Use it for layout, test workflow, and upstream expectations when helping **contributors to [containers/podman-py](https://github.com/containers/podman-py)**—patches, tests, docs, and pull requests for the **Python** bindings to [Podman](https://github.com/containers/podman)’s **REST API**.

Source material: [README.md](https://github.com/containers/podman-py/blob/main/README.md), [CONTRIBUTING.md](https://github.com/containers/podman-py/blob/main/CONTRIBUTING.md), [CODE-OF-CONDUCT.md](https://github.com/containers/podman-py/blob/main/CODE-OF-CONDUCT.md), [SECURITY.md](https://github.com/containers/podman-py/blob/main/SECURITY.md).

- **Audience**: **Upstream podman-py contributors** (or aspiring). This repository is *not* the [Podman](https://github.com/containers/podman) **Go** tree: changes here target `podman/` and tests under `podman/tests` as configured in [pyproject.toml](pyproject.toml). For engine behavior, socket paths, or the HTTP API *protocol*, defer to the Podman project and the [libpod API reference](https://docs.podman.io/en/latest/_static/api.html).
- **Mental model**: The published package is **`podman` on PyPI** ([pyproject.toml](https://github.com/containers/podman-py/blob/main/pyproject.toml) name `podman`); it calls **Podman’s libpod service** over the HTTP API. Clients use **`PodmanClient`**, transport lives under **`podman.api`**, and high-level managers live under **`podman.domain`**. Distinguish this from the **Go** [stable bindings](https://github.com/containers/podman/tree/main/pkg/bindings) in the `podman` repository when suggesting imports or file paths.
- **Upstream contract**: The library must follow the same HTTP contract as the daemon. Wrong paths, query parameters, or response handling here surface as bugs in **podman-py** even when the daemon is correct. Cross-check [libpod / Podman API](https://docs.podman.io/en/latest/_static/api.html) and the [Podman Reference](https://podman.readthedocs.io/en/latest/Reference.html).
- **Quality bar**: [CONTRIBUTING.md](CONTRIBUTING.md) — **Python >= 3.9**; **ruff** and [CONTRIBUTING’s **pylint**](https://github.com/containers/podman-py/blob/main/CONTRIBUTING.md) (exceptions need a case); `make lint` and **pre-commit** are **ruff** + **mypy**—add **pylint** if reviewers or CI need it. **Coverage**: CONTRIBUTING says **≥ 85%** to merge; [tox.ini `testenv:coverage`](https://github.com/containers/podman-py/blob/main/tox.ini) currently uses `coverage report --fail-under=80`—**prefer the 85% rule** when the two differ. **integration** tests for large changes. **DCO** **Signed-off-by** (`git commit -s`). **Security**: [SECURITY.md](SECURITY.md) and the [Containers security policy](https://github.com/containers/common/blob/main/SECURITY.md).
- **Community**: [CODE-OF-CONDUCT.md](CODE-OF-CONDUCT.md). Coordination: GitHub [issues/PRs](https://github.com/containers/podman-py); [CONTRIBUTING](https://github.com/containers/podman-py/blob/main/CONTRIBUTING.md) also notes IRC **#podman** on irc.libera.chat.

## Project overview

**podman-py** is a Python package (`pip install podman`) that uses the [Podman (libpod) HTTP API](https://docs.podman.io/en/latest/_static/api.html). Many high-level ideas overlap with the [Docker Engine HTTP API](https://docs.docker.com/engine/api/latest/); the **source of truth** for this project is still **libpod** / Podman ([static API](https://docs.podman.io/en/latest/_static/api.html)). Published docs: [readthedocs](https://podman-py.readthedocs.io/en/latest/).

- **Language / runtime**: Python **3.9+** (`requires-python` in [pyproject.toml](pyproject.toml))
- **Build / packaging**: setuptools, [pyproject.toml](pyproject.toml) + [setup.py](setup.py) / [setup.cfg](setup.cfg) as applicable in-tree
- **Tests**: The default story is **pytest**; `testpaths` is set under **`pyproject.toml` → `[tool.pytest.ini_options]`** (see directory `podman/tests`), usually driven by **tox** ([tox.ini](tox.ini)). The [Makefile](Makefile) also provides **`make unittest` / `make integration`**, which use Python’s [unittest](https://docs.python.org/3/library/unittest.html#test-discovery) with **coverage** (see `--fail-under` / `--omit` in the Makefile; same **80%** report threshold as [tox.ini `testenv:coverage`](https://github.com/containers/podman-py/blob/main/tox.ini) today—**CONTRIBUTING** still states **85%** for merge).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [containers/podman-py](https://github.com/containers/podman-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
