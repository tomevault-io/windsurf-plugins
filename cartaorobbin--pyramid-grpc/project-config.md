---
trigger: always_on
description: This document outlines the development rules and guidelines for the pyramid-grpc project. Following these rules ensures code quality, maintainability, and effective collaboration.
---

# Development Rules

This document outlines the development rules and guidelines for the pyramid-grpc project. Following these rules ensures code quality, maintainability, and effective collaboration.

## Core Development Principles

### 1. Plan Before Coding ⭐
**ALWAYS plan before writing any code.**

- **Think first, code second**: Before touching any code, take time to understand the problem and design the solution
- **Create a clear plan**: Write down what you're going to do, how you're going to do it, and what the expected outcome is
- **Consider edge cases**: Think through potential issues and how to handle them
- **Review existing code**: Understand how your changes fit into the existing codebase
- **Break down complex tasks**: Split large features into smaller, manageable pieces

### 2. Task Management with Planning Files ⭐
**ALWAYS maintain planning files to track what we did.**

- **Organized structure**: Use `planning/` directory for all planning files
- **Active vs Backlog**: Use `planning/general.md` for CURRENT tasks being worked on, `planning/backlog.md` for planned future tasks
- **Task Flow**: Move tasks from `planning/backlog.md` → `planning/general.md` → `planning/done.md` as they progress
- **Focus Management**: Keep `planning/general.md` focused (max 1-3 active task groups)
- **Create/update files**: Before starting any work, document what you plan to do in the appropriate planning file
- **Track progress**: Update the files as you complete tasks
- **Document decisions**: Record important decisions and why they were made
- **Include completion status**: Mark tasks as TODO, IN PROGRESS, DONE, or BLOCKED
- **Reference issues/PRs**: Link to relevant GitHub issues or pull requests

## Code Quality Standards

### Pre-Development Checklist
Before starting any development work:

- [ ] Read and understand the requirement
- [ ] Create or update appropriate planning file with your planned work
- [ ] Add new tasks to `planning/backlog.md`, move to `planning/general.md` when starting active work
- [ ] Check existing issues and PRs for related work
- [ ] Plan your approach and get feedback if needed
- [ ] Ensure your development environment is set up (`make install`)
- [ ] **Plan test strategy**: Identify what tests need to be written/updated

### Development Workflow

1. **Branch Management**
   ```bash
   git checkout -b feature/descriptive-name
   # or
   git checkout -b bugfix/issue-description
   ```

2. **Code Development**
   - Follow PEP 8 style guidelines
   - Write clear, self-documenting code
   - Add docstrings to all functions and classes
   - Include type hints where appropriate

3. **Testing Requirements**
   - Write tests for all new functionality
   - Ensure existing tests pass: `make test`
   - Run full test suite: `tox`
   - Aim for high test coverage
   - never use class base testcase
   - avoid mocking, only use mock as a last resource
   - we can use pytest responses fixture.

4. **Test Validation Before Task Completion ⭐**
   **NEVER mark a task as DONE without running and verifying tests pass.**
   
   - **Always run tests**: Execute relevant test suites before marking tasks complete
   - **Fix failing tests**: Address any test failures before task completion
   - **Verify no regressions**: Run existing test suites to ensure no breaking changes
   - **Update planning files**: Only mark tasks as DONE after successful test validation
   - **Document test results**: Include test pass/fail status in planning files
   
   ```bash
   # Required before marking any task as DONE
   python -m pytest tests/test_relevant_module.py -v  # New functionality tests
   python -m pytest tests/test_existing_module.py -v  # Regression tests
   make test  # Full test suite (when appropriate)
   ```

5. **Code Quality Checks**
   ```bash
   make check  # Run formatters and linters
   ```

### Documentation Standards

- **Docstrings**: Use Google-style docstrings for all public functions and classes
- **Comments**: Write comments for complex logic, not obvious code
- **README updates**: Update README.md if adding new features or changing setup
- **Type hints**: Use type hints for function parameters and return values

### Commit Standards

- **Commit messages**: Use clear, descriptive commit messages
- **Atomic commits**: Each commit should represent one logical change
- **Pre-commit hooks**: Let pre-commit hooks format your code automatically

Example commit message format:
```
feat: add user authentication system

- Implement JWT token authentication
- Add user login/logout endpoints
- Include password hashing utilities
- Add authentication middleware

Closes #123
```

## Pull Request Guidelines

### Before Creating a PR

- [ ] Update appropriate planning file with completed work
- [ ] **All tests pass locally** (new functionality + regression tests)
- [ ] **Test results documented** in planning files
- [ ] Code quality checks pass (`make check`)
- [ ] Documentation is updated if needed
- [ ] Commit messages are clear and descriptive

### PR Requirements

1. **Title**: Clear, descriptive title
2. **Description**: 
   - What was changed and why
   - Link to related issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cartaorobbin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
