---
trigger: always_on
description: Test creation conventions for the notebooks repository — pytest patterns, testcontainers, markers, fixtures, allure, and test organization.
---


# Test Conventions for opendatahub-io/notebooks

## Test Types Overview

| Type | Location | Command | Requires |
|------|----------|---------|----------|
| Static / manifest | `tests/test_*.py` | `make test` | Nothing (no container engine) |
| Unit | `tests/unit/` | `make test-unit` | Nothing |
| Container integration | `tests/containers/` | `make test-integration PYTEST_ARGS="--image=<img>"` | Podman/Docker |
| Browser E2E | `tests/browser/` | `npx playwright test` (from `tests/browser/`) | Node, Playwright |
| K8s notebook smoke | `make test-<notebook>` | `scripts/test_jupyter_with_papermill.sh` | kubectl, deployed workbench |
| Go | `scripts/buildinputs/` | `make test-unit` | Go toolchain |

## Framework and Tools

| Tool | Purpose |
|------|---------|
| pytest | Test runner for all Python tests |
| pytest-subtests | Granular sub-assertions within a single test |
| pytest-cov | Coverage (XML + terminal) |
| allure-pytest | Issue tracking + step decoration |
| testcontainers | Container lifecycle for integration tests |
| docker (Python) | Low-level container operations |
| pyfakefs | Filesystem mocking for unit tests |
| Playwright | Browser tests (TypeScript) |
| papermill | Notebook execution verification |

## File Naming and Location

- Static/manifest tests: `tests/test_*.py` (module-level functions)
- Unit tests: `tests/unit/<mirror-source-path>/test_*.py`
- Container tests: `tests/containers/**/*_test.py` (class-based)
- Browser tests: `tests/browser/tests/*.spec.ts`

## Test Organization

### Static tests (`tests/test_*.py`)

Use **module-level functions**. Group related assertions with `subtests`:

```python
from __future__ import annotations

import pytest
import pytest_subtests

from tests import PROJECT_ROOT


def test_something_across_manifests(subtests: pytest_subtests.SubTests):
    for file in PROJECT_ROOT.glob("**/pyproject.toml"):
        with subtests.test(msg=str(file.relative_to(PROJECT_ROOT))):
            # assertion per file
            assert condition, f"Failed for {file}"
```

### Container tests (`tests/containers/**/*_test.py`)

Use **`Test*` classes** with typed fixture parameters:

```python
from __future__ import annotations

import allure
import pytest

from tests.containers import conftest, docker_utils
from tests.containers.workbenches.workbench_image_test import WorkbenchContainer


class TestMyFeature:
    def test_something(
        self,
        workbench_image: conftest.Image,
        subtests: pytest_subtests.SubTests,
    ):
        with WorkbenchContainer(workbench_image) as container:
            container.start()
            ecode, output = container.exec(["command", "arg"])
            assert ecode == 0, output.decode()
```

## Markers

All markers must be registered in `pytest.ini` (strict mode). Current markers:

| Marker | Purpose | Excluded from default integration run |
|--------|---------|--------------------------------------|
| `openshift` | Needs live OpenShift cluster | Yes |
| `cuda` | Needs NVIDIA GPU | Yes |
| `rocm` | Needs AMD GPU | Yes |
| `manifest_validation` | Slow registry/skopeo checks | Yes |
| `buildonlytest` | Only run during build CI, not `make test` | Yes (different filter) |

Usage:

```python
@pytest.mark.openshift
def test_image_run_on_openshift(self, ...):
    ...
```

## Parametrize Patterns

### Directory/matrix parametrize

```python
@pytest.mark.parametrize("manifests_directory", [
    manifests.MANIFESTS_ODH_DIR,
    manifests.MANIFESTS_RHOAI_DIR,
])
def test_image_manifests(manifests_directory: Path, subtests):
    ...
```

### Session-scoped image parametrize (via `--image` CLI)

Do NOT use `@pytest.mark.parametrize` for the `image` fixture — it's handled by `pytest_generate_tests` in `tests/containers/conftest.py` with `scope="session"`. Just declare `image: str` as a parameter.

## Fixture Conventions

### Session-scoped image chain

Fixtures skip automatically based on image labels:

```python
def test_jupyterlab_feature(self, jupyterlab_image: conftest.Image):
    # Auto-skips if image is not a JupyterLab workbench
    ...
```

Available session fixtures: `image`, `workbench_image`, `jupyterlab_image`, `datascience_image`, `codeserver_image`, `runtime_image`, `cuda_image`, `rocm_image`, `container_arch`.

### Function-scoped fixtures

- `tf` — `TestFrame` for Kubernetes resource management
- `test_frame` — local resource list with deferred cleanup
- `subtests` — from pytest-subtests plugin (always available)

## Testcontainers Patterns

### Container lifecycle

```python
from tests.containers.docker_utils import running_container, NotebookContainer, BestEffortCleanup

# Short-lived check
with running_container(image) as container:
    ecode, output = container.exec(["rpm", "-qa"])
    assert ecode == 0

# Long-lived with WorkbenchContainer (HTTP readiness)
with WorkbenchContainer(workbench_image) as container:
    container.start(wait_for_readiness=True)
    resp = requests.get(container.base_url)
    assert resp.status_code == 200
```

### Exec and file operations

```python
# Execute command
ecode, output = container.exec(["/bin/sh", "-c", "echo hello"])
assert ecode == 0, output.decode()

# Copy file into container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatahub-io/notebooks](https://github.com/opendatahub-io/notebooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
