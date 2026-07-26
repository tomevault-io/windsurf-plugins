---
trigger: always_on
description: Always source the project's virtual environment before running any commands:
---

# Agent Guidelines for Terrakit Development

## Environment Setup

### Virtual Environment
Always source the project's virtual environment before running any commands:

```bash
source .venv/bin/activate
```

For commands that need the virtual environment, use:
```bash
source .venv/bin/activate && <command>
```

## Test-Driven Development (TDD)

### Core Principles

1. **Write Tests First**: Before implementing any feature or fix, write the test that defines the expected behavior
2. **Red-Green-Refactor Cycle**:
   - **Red**: Write a failing test
   - **Green**: Write minimal code to make the test pass
   - **Refactor**: Improve code while keeping tests green

### Testing Structure

The project follows a structured testing approach:

```
tests/
├── component_tests/          # Unit and component-level tests
│   ├── chip/                # Tiling functionality tests
│   ├── download/            # Download functionality tests
│   │   └── data_connectors/ # Individual connector tests
│   ├── general_utils/       # Utility function tests
│   ├── store/               # Storage functionality tests
│   └── transform/           # Transformation tests
└── integration_tests/        # End-to-end integration tests
```

### Test Development Workflow

#### 1. Before Making Changes
- Read existing tests to understand current behavior
- Identify what needs to be tested
- Check test coverage for the area you're modifying

#### 2. Writing Tests
```bash
# Run tests to ensure current state
source .venv/bin/activate && pytest tests/component_tests/<module>/ -v

# Write your test first
# Example: tests/component_tests/download/test_new_feature.py
```

#### 3. Test Naming Conventions
- Test files: `test_<module_name>.py`
- Test functions: `test_<specific_behavior>()`
- Use descriptive names that explain what is being tested

#### 4. Running Tests

Run all tests:
```bash
source .venv/bin/activate && pytest
```

Run specific test file:
```bash
source .venv/bin/activate && pytest tests/component_tests/download/test_download_data.py -v
```

Run specific test:
```bash
source .venv/bin/activate && pytest tests/component_tests/download/test_download_data.py::test_specific_function -v
```

Run with coverage:
```bash
source .venv/bin/activate && pytest --cov=terrakit --cov-report=html
```

#### 5. Test Fixtures
- Use `conftest.py` files for shared fixtures
- Keep fixtures close to where they're used
- Document complex fixtures

### Development Process

#### Adding New Features

1. **Understand Requirements**
   - Read related documentation in `docs/`
   - Review existing similar implementations
   - Check `tests/` for existing test patterns

2. **Write Tests First**
   ```bash
   # Create test file
   # Write failing tests that define expected behavior
   source .venv/bin/activate && pytest tests/component_tests/<module>/test_new_feature.py -v
   # Tests should fail (Red)
   ```

3. **Implement Feature**
   - Write minimal code to pass tests
   - Follow existing code patterns
   - Keep implementation simple

4. **Verify Tests Pass**
   ```bash
   source .venv/bin/activate && pytest tests/component_tests/<module>/test_new_feature.py -v
   # Tests should pass (Green)
   ```

5. **Refactor**
   - Improve code quality
   - Ensure tests still pass
   - Add documentation

6. **Run Full Test Suite**
   ```bash
   source .venv/bin/activate && pytest
   ```

#### Fixing Bugs

1. **Write Failing Test**
   - Create a test that reproduces the bug
   - Verify the test fails

2. **Fix the Bug**
   - Implement the fix
   - Ensure the new test passes

3. **Verify No Regressions**
   ```bash
   source .venv/bin/activate && pytest
   ```

#### Modifying Existing Code

1. **Read Existing Tests**
   ```bash
   source .venv/bin/activate && pytest tests/component_tests/<module>/ -v
   ```

2. **Update Tests First**
   - Modify tests to reflect new expected behavior
   - Ensure tests fail appropriately

3. **Update Implementation**
   - Make code changes
   - Verify tests pass

### Code Quality Checks

Before committing, always run:

```bash
# Activate environment
source .venv/bin/activate

# Run tests
pytest

# Run type checking
mypy terrakit/

# Run linting (if configured)
# pre-commit hooks will run automatically on commit
```

### Testing Best Practices

1. **Test Independence**: Each test should be independent and not rely on other tests
2. **Clear Assertions**: Use descriptive assertion messages
3. **Mock External Dependencies**: Use mocks for external APIs, file systems, etc.
4. **Test Edge Cases**: Include tests for boundary conditions and error cases
5. **Keep Tests Fast**: Unit tests should run quickly; use integration tests for slower operations
6. **Maintain Test Data**: Store test resources in `tests/resources/`

### Common Testing Patterns

#### Testing Data Connectors
```python
# See: tests/component_tests/download/data_connectors/
# - Mock external API calls
# - Test parameter validation
# - Test data transformation
# - Test error handling
```

#### Testing Transformations
```python
# See: tests/component_tests/transform/
# - Test with sample data
# - Verify output format
# - Test edge cases (empty data, invalid data)
```

#### Testing Utilities
```python
# See: tests/component_tests/general_utils/
# - Test pure functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [torchgeo/terrakit](https://github.com/torchgeo/terrakit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
