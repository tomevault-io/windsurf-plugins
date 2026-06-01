---
trigger: always_on
description: LLM Sandbox is a lightweight and portable sandbox environment designed to run Large Language Model (LLM) generated code in a safe and isolated mode. The project provides secure execution environments for AI-generated code with:
---

# GitHub Copilot Instructions for LLM Sandbox

## Project Overview

LLM Sandbox is a lightweight and portable sandbox environment designed to run Large Language Model (LLM) generated code in a safe and isolated mode. The project provides secure execution environments for AI-generated code with:

- **Multi-language support**: Python, JavaScript/Node.js, Java, C++, Go, R, and Ruby
- **Flexible backends**: Docker, Kubernetes, Podman, and Micromamba
- **Security-first design**: Isolated execution, security policies, resource limits, and network isolation
- **LLM framework integration**: Works with LangChain, LangGraph, LlamaIndex, OpenAI, and more
- **Model Context Protocol (MCP)**: Server implementation for MCP clients like Claude Desktop

## Code Style and Formatting

### Python Code Standards

- **Python version**: Support Python 3.10, 3.11, 3.12, and 3.13
- **Style guide**: Follow PEP 8 with Ruff enforced rules (see `pyproject.toml`)
- **Line length**: Maximum 120 characters
- **Type hints**: Always use type hints for function signatures and class attributes
- **Docstrings**: Use Google-style docstrings for all public functions, classes, and modules

Example:
```python
def execute_code(code: str, timeout: int = 30) -> ExecutionResult:
    """Execute code in the sandbox environment.

    Args:
        code: The code to execute
        timeout: Maximum execution time in seconds

    Returns:
        ExecutionResult containing stdout, stderr, and exit code

    Raises:
        TimeoutError: If execution exceeds timeout
        SecurityError: If code violates security policy
    """
    pass
```

### Code Organization

- Use Pydantic models for data structures and configuration
- Follow existing module structure:
  - `llm_sandbox/core/`: Core functionality and base classes
  - `llm_sandbox/language_handlers/`: Language-specific handlers
  - `llm_sandbox/mcp_server/`: MCP server implementation
  - Backend implementations: `docker.py`, `kubernetes.py`, `podman.py`, `micromamba.py`

### Import Style

```python
# Standard library imports
from typing import Any
from pathlib import Path

# Third-party imports
from pydantic import BaseModel, Field

# Local imports
from llm_sandbox.const import SupportedLanguage
from llm_sandbox.exceptions import SandboxError
```

## Testing Practices

### Test Organization

- All tests in `tests/` directory
- Test file naming: `test_<module_name>.py`
- Use pytest for all tests
- Organize tests into classes when testing multiple aspects of a feature

### Test Structure

Follow the Arrange-Act-Assert pattern:

```python
def test_feature_description() -> None:
    """Test description explaining what is being tested."""
    # Arrange
    session = SandboxSession(lang="python")
    code = "print('hello')"

    # Act
    result = session.run(code)

    # Assert
    assert result.stdout == "hello\n"
    assert result.exit_code == 0
```

### Test Fixtures and Mocking

- Use `conftest.py` for shared fixtures
- Mock external dependencies (Docker, Kubernetes, etc.) in unit tests
- Use `@patch` decorator for mocking:

```python
@patch("llm_sandbox.session.find_spec")
@patch("llm_sandbox.docker.docker.from_env")
def test_with_mocks(mock_docker: MagicMock, mock_find_spec: MagicMock) -> None:
    """Test with mocked dependencies."""
    mock_find_spec.return_value = MagicMock()
    # Test implementation
```

### Parametrized Tests

Use `@pytest.mark.parametrize` for testing multiple inputs:

```python
@pytest.mark.parametrize("language,expected_extension", [
    ("python", "py"),
    ("javascript", "js"),
    ("java", "java"),
])
def test_language_extensions(language: str, expected_extension: str) -> None:
    """Test language handler extensions."""
    handler = LanguageHandlerFactory.create_handler(language)
    assert handler.file_extension == expected_extension
```

### Coverage Expectations

- Maintain high test coverage (aim for >80%)
- Test both success and error cases
- Include edge cases and boundary conditions
- Security tests must not make real Docker connections (use `mock_docker_backend` fixture)

## Security Considerations

### Security Policy

The project implements a comprehensive security scanning system:

- **Pattern-based detection**: Identify dangerous code patterns
- **Module restrictions**: Block imports of restricted modules
- **Severity levels**: `SAFE`, `LOW`, `MEDIUM`, `HIGH`
- **Security threshold**: Configurable blocking threshold

When adding security features:

```python
# Define security patterns
pattern = SecurityPattern(
    pattern=r"import\s+os",
    description="Direct OS module access",
    severity=SecurityIssueSeverity.MEDIUM,
)

# Add to security policy
policy = SecurityPolicy(
    severity_threshold=SecurityIssueSeverity.MEDIUM,
    patterns=[pattern],
)
```

### Container Security

- All code execution happens in isolated containers
- Resource limits (CPU, memory, execution time) are enforced
- Network isolation can be configured
- File system access is restricted

### Best Practices

- Never trust user-provided code without security scanning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vndee/llm-sandbox](https://github.com/vndee/llm-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
