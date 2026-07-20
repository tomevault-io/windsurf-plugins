---
trigger: always_on
description: Guidelines for testing Langflow components and backend code, with emphasis on async patterns and robust, well-documented testing practices.
---


# Testing Guidelines for Langflow

## Purpose
Guidelines for testing Langflow components and backend code, with emphasis on async patterns and robust, well-documented testing practices.

---

## 1. Testing Structure

### Backend Tests Location
- **Unit Tests:** `src/backend/tests/`
- **Component Tests:** `src/backend/tests/unit/components/` (organized by component subdirectory)
- **Integration Tests:** Available via `make integration_tests` (requires additional setup)

### Test File Naming
- Use same filename as component with appropriate test prefix/suffix
- Example: `my_component.py` → `test_my_component.py`

---

## 2. Built-in Fixtures & Base Classes

### `client` Fixture (FastAPI Test Client)
- Defined in `src/backend/tests/conftest.py`
- Provides an **async** `httpx.AsyncClient` connected to the full application via `ASGITransport` + `LifespanManager`.
- Use it for API tests:
  ```python
  async def test_login_endpoint(client):
      response = await client.post("api/v1/login", data={"username": "foo", "password": "bar"})
      assert response.status_code == 200
  ```
- Automatically configured with an **in-memory SQLite database** and mocked environment variables. No additional setup needed in individual tests.
- Skip client creation by marking the test with `@pytest.mark.noclient`.

### `ComponentTestBase` Family
Located in `src/backend/tests/base.py`.

| Base Class | Creates `client`? | Typical Use | Notes |
|------------|------------------|-------------|-------|
| `ComponentTestBase` | No | Core logic for component version testing | Requires you to provide fixtures described below. |
| `ComponentTestBaseWithClient` | Yes (`@pytest.mark.usefixtures("client")`) | Components that need API access during `run()` | Inherit when the component interacts with backend services. |
| `ComponentTestBaseWithoutClient` | No | Pure-logic components with no API calls | Lightweight alternative. |

Required fixtures for subclasses:
1. `component_class` → the component **class** under test.
2. `default_kwargs` → dict of kwargs to instantiate the component (can be empty).
3. `file_names_mapping` → list of `VersionComponentMapping` dicts mapping historical **Langflow versions** to module/file names.

Example skeleton:
```python
from tests.base import ComponentTestBaseWithClient, VersionComponentMapping, DID_NOT_EXIST
from langflow.components.my_namespace import MyComponent

class TestMyComponent(ComponentTestBaseWithClient):
    @pytest.fixture
    def component_class(self):
        return MyComponent

    @pytest.fixture
    def default_kwargs(self):
        return {"foo": "bar"}

    @pytest.fixture
    def file_names_mapping(self):
        return [
            VersionComponentMapping(version="1.1.1", module="my_module", file_name="my_component.py"),
            # Older versions can be mapped or DID_NOT_EXIST
            VersionComponentMapping(version="1.0.19", module="my_module", file_name=DID_NOT_EXIST),
        ]
```

`ComponentTestBase` automatically provides:
- `test_latest_version` → Instantiates component via `component_class` and asserts `run()` doesn't return `None`.
- `test_all_versions_have_a_file_name_defined` → Ensures mapping completeness vs `SUPPORTED_VERSIONS` constant (`src/backend/tests/constants.py`).
- `test_component_versions` (parametrised) → Builds component from source for each supported version and asserts successful execution.

When adding a new component test, **inherit from the correct base class and provide the three fixtures**. This greatly reduces boilerplate and enforces version compatibility.

---

## 3. Component Testing Requirements

### Minimum Testing Requirements
- **Unit Tests:** Create comprehensive unit tests for all new components
- **Manual Test Documentation:** If unit tests are incomplete, create a Markdown file with manual testing steps
  - Location: Same directory as unit tests
  - Filename: Same as component but with `.md` extension
  - Content: Detailed manual testing steps and expected outcomes

### Testing Best Practices
- Test both sync and async code paths
- Mock external dependencies appropriately
- Test error handling and edge cases
- Validate input/output behavior
- Test component initialization and configuration

---

## 4. Async Testing Patterns

### Async Component Testing
```python
import pytest
import asyncio

@pytest.mark.asyncio
async def test_async_component():
    # Test async component methods
    result = await component.async_method()
    assert result is not None
```

### Testing Background Tasks
```python
@pytest.mark.asyncio
async def test_background_task_completion():
    # Ensure background tasks complete properly
    task = asyncio.create_task(component.background_operation())
    result = await asyncio.wait_for(task, timeout=5.0)
    assert result.success
```

### Testing Queue Operations
```python
@pytest.mark.asyncio
async def test_queue_operations():
    # Test queue put/get operations without blocking
    queue = asyncio.Queue()

    # Non-blocking put
    queue.put_nowait(test_data)

    # Verify queue processing
    result = await asyncio.wait_for(queue.get(), timeout=1.0)
    assert result == test_data
```

---

## 5. Special Testing Considerations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UhtredRegress/LangFlowClone](https://github.com/UhtredRegress/LangFlowClone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
