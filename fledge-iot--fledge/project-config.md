---
trigger: always_on
description: Python unit testing rules for Fledge - test organization, framework, execution, and coverage
---


# Python Unit Testing Guidelines

## Test Organization & Structure

### Directory Structure
- **Unit Tests**: Located in [tests/unit/python/](mdc:tests/unit/python/) 
- **Test Instructions**: Follow detailed guidelines in [tests/README.rst](mdc:tests/README.rst)
- **File Structure**: Tests should mirror the component structure under `tests/unit/python/fledge/<component>`
- **Template**: Use [tests/unit/python/__template__.py](mdc:tests/unit/python/__template__.py) as starting point

### Test File Conventions
- **Naming**: Test files must begin with `test_` for pytest auto-discovery
- **Pattern**: `test_<module_name>.py`
- **Location**: Place tests in correct directory matching component structure
- **Imports**: Follow Fledge import patterns and avoid circular dependencies
- **Docstrings**: Include Pydoc-compatible docstrings for test classes and methods

### Test Class & Method Organization
- Group related tests in classes using `TestClassName` pattern
- Use descriptive test method names: `test_should_return_success_when_valid_input`
- Organize tests logically: happy path, edge cases, error conditions
- Use pytest fixtures for common setup and teardown
- Keep tests focused and atomic - one assertion per test when possible

## Testing Framework & Dependencies

### Primary Framework
- **Framework**: pytest (version specified in [python/requirements-test.txt](mdc:python/requirements-test.txt))
- **Dependencies**: All testing dependencies are managed in requirements-test.txt
- **Dependency Management**: Reference requirements-test.txt for current versions - do not hardcode versions in documentation

### Core Testing Dependencies
Key testing packages (see [python/requirements-test.txt](mdc:python/requirements-test.txt) for current versions):
- `pytest` - Main testing framework
- `pytest-asyncio` - For async testing support
- `pytest-mock` - Mocking framework integration
- `pytest-cov` - Code coverage reporting
- `pytest-aiohttp` - aiohttp testing utilities
- `pylint` - Code quality and linting

### Additional Testing Dependencies
- `requests` - For HTTP client testing
- `pyserial` - For RTU serial testing
- `pytz` - Timezone handling in tests
- `aiohttp` and `yarl` - Keep versions synchronized with main requirements

## Test Configuration

### pytest Configuration
- **Configuration File**: [tests/unit/python/.pytest.ini](mdc:tests/unit/python/.pytest.ini)
- **Minimum Version**: Check requirements-test.txt for current pytest version
- **Excluded Directories**: Plugin directories excluded from test recursion
- **Test Discovery**: Automatic discovery of test_*.py files

### Coverage Configuration
- **Configuration File**: [tests/unit/python/.coveragerc](mdc:tests/unit/python/.coveragerc)
- **Omitted Files**: 
  - `__init__.py` and `__template__.py` files
  - Setup files and plugin directories
  - Test directories themselves
- **Coverage Scope**: Focus on core Fledge components, exclude plugin frameworks

## Test Execution

### Basic pytest Commands
Refer to [tests/README.rst](mdc:tests/README.rst) for complete instructions:

```bash
# Execute all tests in specific file
pytest test_filename.py

# Execute specific test class
pytest test_filename.py::TestClass

# Execute specific test method
pytest test_filename.py::TestClass::test_case

# Verbose output with detailed information
pytest -s -vv

# Run tests with coverage
pytest --cov=. --cov-report=html
```

### Advanced Test Execution
```bash
# Run tests with full coverage report
pytest -s -vv tests/unit/python/fledge/ --cov=. --cov-report=html --cov-config tests/unit/python/.coveragerc

# Run tests with XML coverage for CI/CD
pytest --cov=. --cov-report html:coverage_html --cov-report xml:coverage.xml

# Run specific test patterns
pytest -k "test_pattern_name"

```

## Code Coverage

### Coverage Configuration
- **Tool**: pytest-cov framework integration
- **Config File**: [tests/unit/python/.coveragerc](mdc:tests/unit/python/.coveragerc)
- **Output Formats**: HTML, XML, and terminal reports
- **Exclusions**: Configured to omit template files, plugins, and test directories

### Coverage Commands

#### Basic Coverage Reports
```bash
# Terminal coverage report (default)
pytest --cov=. --cov-report=term

# Terminal with missing lines shown
pytest --cov=. --cov-report=term-missing

# HTML coverage report (recommended for development)
pytest --cov=. --cov-report=html

# JSON coverage report for tools integration
pytest --cov=. --cov-report=json

# XML coverage report for CI/CD systems
pytest --cov=. --cov-report=xml
```

#### Comprehensive Coverage Commands
```bash
# Full coverage with HTML and XML (for CI/CD)
pytest --cov=. --cov-report=html:coverage_html --cov-report=xml:coverage.xml --cov-config=tests/unit/python/.coveragerc

# Coverage with specific source directory and custom config
pytest tests/unit/python/fledge/ --cov=fledge --cov-report=html --cov-config=tests/unit/python/.coveragerc

# Coverage with minimum percentage threshold (fail if below)
pytest --cov=. --cov-report=term --cov-fail-under=80

# Coverage with detailed terminal output and HTML
pytest --cov=. --cov-report=term-missing --cov-report=html:htmlcov

# Coverage for specific modules only
pytest --cov=fledge.services.core --cov=fledge.common --cov-report=html

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fledge-iot/fledge](https://github.com/fledge-iot/fledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
