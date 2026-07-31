---
trigger: always_on
description: - **Analyze Existing Tests:** Before writing new tests, study `tests/test_sync.py`, `tests/test_config.py`, and other relevant test files to understand existing patterns, fixture usage, and assertion styles.
---

# Ground Rules for Writing vcspull Tests

## 1. Study First, Be Homogenous
- **Analyze Existing Tests:** Before writing new tests, study `tests/test_sync.py`, `tests/test_config.py`, and other relevant test files to understand existing patterns, fixture usage, and assertion styles.
- **Maintain Consistency:** Strive for homogeneity in test structure, naming conventions, and overall style.
- **Project Conventions:** Adhere to project-specific conventions like using `typing as t` and `pathlib.Path` for all path manipulations.

## 2. Fixture Prioritization and Usage
- **`libvcs` Fixtures First:** For any VCS-related operations (creating repos, committing, checking status), **always** prioritize using fixtures from `libvcs.pytest_plugin` (available via `vcspull/conftest.py`). Examples:
    - `create_git_remote_repo`, `create_svn_remote_repo`, `create_hg_remote_repo` (and their `mercurial`/`subversion` counterparts if using those directly) for setting up test repositories.
    - `git_repo`, `svn_repo`, `hg_repo` for pre-configured `Sync` objects.
    - `git_commit_envvars` for environment variables needed for git commits.
- **Pytest Built-in Fixtures:** Utilize standard `pytest` fixtures like `tmp_path` for temporary files and directories.
- **Custom Project Fixtures:**
    - For common non-VCS setup (e.g., mocked home/CWD, config file setup), use or create well-defined fixtures.
    - Place shared fixtures in `vcspull/conftest.py` or `vcspull/tests/conftest.py`. Module-specific fixtures can reside in the test file itself.
    - Example: `home_path`, `cwd_path` (refactored to use `monkeypatch`), `setup_teardown_test_config_dir`.
- **`autouse=True`:** Use sparingly, only for fixtures that genuinely apply to *all* tests within their scope.

## 3. Mocking Strategy: `monkeypatch` vs. `mocker`
- **`monkeypatch` (pytest built-in):**
    - **Environment & Globals:** Use for modifying global settings, environment variables (`monkeypatch.setenv()`, `monkeypatch.delenv()`), the current working directory (`monkeypatch.chdir()`), or `sys.path`.
    - **Patching Attributes/Builtins:** Use `monkeypatch.setattr()` to modify attributes of classes/objects (e.g., `pathlib.Path.home`) or to replace functions/methods in external libraries or Python builtins. When needing to control the home directory, prefer using established project fixtures like `user_path`, `home_path`, or `config_path`. These fixtures are responsible for correctly mocking `pathlib.Path.home()` internally, typically using `monkeypatch.setattr()`. Avoid direct `monkeypatch.setattr(pathlib.Path, "home", ...)` in individual tests if a suitable project fixture exists.
    - **Dictionary Items:** Use `monkeypatch.setitem()` and `monkeypatch.delitem()` for modifying dictionaries.
    - Refer to [Pytest Monkeypatch Documentation](mdc:.dot-config/https:/docs.pytest.org/en/stable/how-to/monkeypatch.html).
- **`mocker` (from `pytest-mock`):**
    - **Application Code:** Primarily use for patching functions, methods, or objects *within the `vcspull` application code itself* (e.g., `mocker.patch('vcspull.cli.add.some_function')`).
    - **Assertions:** Use `mocker` when you need to assert how a mock was called, its return values, or to simulate side effects for your application's internal logic.
- **Clarity in Mocking (CRITICAL):**
    - For **every** use of `mocker.patch()`, `mocker.patch.object()`, `monkeypatch.setattr()`, `monkeypatch.setenv()`, etc., include comments explaining:
        - **`# WHAT:`**: What specific function, method, attribute, or environment variable is being simulated or altered.
        - **`# WHY:`**: The reason for the mock – what behavior is being controlled or isolated for the test's purpose.

## 4. Test Structure and Assertions
- **Atomic Tests:** Each test function should verify a single, specific piece of functionality or scenario.
- **Clear Naming:** Test functions and fixtures should have descriptive names (e.g., `test_add_repo_new_config_cwd`).
- **Docstrings:** Test functions should have a concise docstring explaining what is being tested.
- **Plain Assertions:** Use standard `assert` statements for verifications.
- **Logging:** Use the `caplog` fixture to assert specific log messages when testing command output or internal logging.
- **Error Handling:** Explicitly test for expected exceptions using `pytest.raises()`.

### Parameterized Test Structure
For tests involving multiple scenarios managed by `@pytest.mark.parametrize`, use `typing.NamedTuple` to define the structure of each test case. This promotes readability and consistency.
- Include a `test_id: str` field in the `NamedTuple` for clear test identification in pytest output.
- Define a list of these `NamedTuple` instances for your test scenarios.
- Use `pytest.mark.parametrize` with `ids=lambda tc: tc.test_id` (or similar) for descriptive test names.

```python
# Example of Parameterized Test Structure
import typing as t
import pytest

class MyTestScenario(t.NamedTuple):
    test_id: str
    input_arg: str
    expected_output: str
    # ... other relevant parameters

TEST_SCENARIOS: list[MyTestScenario] = [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vcs-python/vcspull](https://github.com/vcs-python/vcspull) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
