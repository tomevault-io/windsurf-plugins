---
trigger: always_on
description: This document provides guidelines for AI agents working on the Jazzband.co codebase.
---

# AI Agent Guidelines for Jazzband.co

This document provides guidelines for AI agents working on the Jazzband.co codebase.

## Testing Patterns

### Test Framework

This project uses **pytest** with the following conventions:

- ✅ **Use pytest-mock's `mocker` fixture** - Available automatically via `pytest-mock` plugin
- ✅ **Use `mocker.patch()` for patching** - Automatically cleaned up after each test
- ✅ **Use `monkeypatch` fixture for simple attribute/env changes** - When full mocking isn't needed
- ✅ **Function-based tests** - Not class-based test methods
- ❌ **Never import `unittest.mock` or `mock`** - Use `mocker` fixture instead
- ❌ **No `@patch` decorators** - Use `mocker.patch()` instead
- ❌ **No `with patch()` context managers** - Use `mocker.patch()` instead

### Test Structure

```python
def test_function_name(fixture1, fixture2, mocker):
    """Clear docstring describing what's being tested."""
    # Arrange - Set up test data and mocks
    mock_obj = mocker.MagicMock()
    mock_obj.method.return_value = expected_value
    mock_something = mocker.patch("module.path.to.object")

    # Act - Call the function under test
    result = function_under_test(args)

    # Assert - Verify the results
    assert result == expected_value
    mock_obj.method.assert_called_once()
```

### Available Fixtures

Common fixtures are defined in `tests/conftest.py`:

#### Application Fixtures
- `app` - Flask application instance
- `test_app_context` - Application context for isolated tests

#### Mock Fixtures
- `mocker` - pytest-mock's mocker fixture (use this instead of unittest.mock)
- `github_blueprint` - Returns `(blueprint, mock_admin_session)` tuple
- `mock_user` - Mock user object with id, login, email
- `mock_project` - Mock project instance
- `mock_github_api` - Mock GitHub API client
- `mock_response_factory` - Factory for creating mock HTTP responses
- `create_mock_response(status_code, data)` - Helper to create mock responses

#### Test Data Fixtures
- `github_org_name` - Test organization name ("test-org-name")
- `test_project_name` - Test project name ("test-project")

### Mocking Patterns

#### Mocking Database Queries

```python
def test_with_database_query(mocker):
    """Test function that queries the database."""
    mock_user = mocker.MagicMock()
    mock_user.login = "test-user"

    mock_user_class = mocker.patch("module.path.User")
    mock_user_class.query.get.return_value = mock_user

    # Your test code here
    result = function_that_queries_user(user_id=123)

    # Verify the query was called
    mock_user_class.query.get.assert_called_once_with(123)
```

#### Mocking GitHub API Calls

```python
def test_github_api_call(mocker, test_app_context):
    """Test function that calls GitHub API."""
    mock_github = mocker.patch("jazzband.projects.tasks.github")

    # Setup mock response
    mock_response = mocker.MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"success": True}
    mock_github.some_method.return_value = mock_response

    # Call function
    result = function_that_uses_github_api()

    # Verify
    mock_github.some_method.assert_called_once()
    assert result is not None
```

#### Mocking Multiple Related Objects

```python
def test_with_multiple_mocks(mocker):
    """Test with multiple related mock objects."""
    # Create related mocks
    mock_user = mocker.MagicMock()
    mock_user.login = "test-lead"

    mock_project = mocker.MagicMock()
    mock_project.team_slug = "test-project"
    mock_project.leads_team_slug = "test-project-leads"

    # Patch multiple classes using mocker.patch
    mock_user_class = mocker.patch("module.User")
    mock_project_class = mocker.patch("module.Project")
    mock_user_class.query.get.return_value = mock_user
    mock_project_class.query.get.return_value = mock_project

    # Test code here
```

### Testing GitHub API Methods

When testing methods on `GitHubBlueprint`:

```python
def test_github_method(github_blueprint, github_org_name, mocker):
    """Test a GitHub API method."""
    blueprint, mock_admin_session = github_blueprint

    # Setup mock response
    mock_response = mocker.MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"id": 12345, "name": "test"}
    mock_admin_session.get.return_value = mock_response

    # Call the method being tested
    result = GitHubBlueprint.method_name(blueprint, "argument")

    # Verify API was called correctly
    mock_admin_session.get.assert_called_once_with("expected/path")

    # Verify the result
    assert result.status_code == 200
    assert result.json()["id"] == 12345
```

### Testing Task Functions

Task functions often interact with database models and external APIs:

```python
def test_task_function(mocker, test_app_context):
    """Test a Spinach task function."""
    user_id = 123
    project_id = 456

    # Setup mocks for database objects
    mock_user = mocker.MagicMock()
    mock_user.login = "test-user"

    mock_project = mocker.MagicMock()
    mock_project.name = "test-project"

    # Patch database and GitHub using mocker.patch
    mock_user_class = mocker.patch("jazzband.projects.tasks.User")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jazzband/website](https://github.com/jazzband/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
