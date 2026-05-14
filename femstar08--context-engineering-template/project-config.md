---
trigger: always_on
description: - **Always read `PLANNING.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
---

### 🔄 Enhanced Project Awareness & Context Engineering
- **Always read `PLANNING.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
- **Check `TASK.md`** before starting a new task. If the task isn't listed, add it with a brief description and today's date.
- **Use enhanced Context Engineering system** with failure pattern awareness and validation loops.
- **Use consistent naming conventions, file structure, and architecture patterns** as described in `PLANNING.md`.
- **Use venv_linux** (the virtual environment) whenever executing Python commands, including for unit tests.

### 🧠 Context Engineering Enhanced Rules
- **Always validate PRPs before execution** using `validate-prp` command to catch issues early.
- **Use failure pattern awareness** from the knowledge base to prevent common mistakes.
- **Follow multi-level validation** approach: syntax → unit tests → integration → performance.
- **Run post-implementation analysis** to capture learnings and improve future implementations.
- **Update knowledge base** with new patterns and metrics after each implementation.

### 🧱 Code Structure & Modularity
- **Never create a file longer than 500 lines of code.** If a file approaches this limit, refactor by splitting it into modules or helper files.
- **Organize code into clearly separated modules**, grouped by feature or responsibility.
  For agents this looks like:
    - `agent.py` - Main agent definition and execution logic 
    - `tools.py` - Tool functions used by the agent 
    - `prompts.py` - System prompts
- **Use clear, consistent imports** (prefer relative imports within packages).
- **Use python_dotenv and load_env()** for environment variables.
- **Always implement proper error handling** with specific exception types and meaningful error messages.

### 🧪 Testing & Reliability Enhanced
- **Always create Pytest unit tests for new features** (functions, classes, routes, etc).
- **Follow test-driven development** when implementing complex features.
- **After updating any logic**, check whether existing unit tests need to be updated. If so, do it.
- **Tests should live in a `/tests` folder** mirroring the main app structure.
  - Include at least:
    - 1 test for expected use
    - 1 edge case
    - 1 failure case
    - 1 async context test (if applicable)
- **Use proper test isolation** to prevent test pollution and ensure consistent results.
- **Mock external dependencies** appropriately but avoid over-mocking.

### ⚡ Performance & Quality Standards
- **Always use async/await consistently** - never mix sync and async contexts.
- **Implement proper connection pooling** for database and external API connections.
- **Use connection timeouts** for all external API calls.
- **Implement retry logic with exponential backoff** for transient failures.
- **Monitor memory usage** and implement proper cleanup for long-running processes.
- **Use proper type hints** throughout the codebase for better maintainability.

### 🔧 Validation & Quality Assurance
- **Run ruff check and fix** before committing any code.
- **Run mypy for type checking** and fix all type errors.
- **Run security scanning with bandit** for security vulnerabilities.
- **Ensure test coverage is above 80%** for new features.
- **Use pre-commit hooks** if available to enforce quality standards.

### ✅ Task Completion Enhanced
- **Mark completed tasks in `TASK.md`** immediately after finishing them.
- **Run post-implementation analysis** using `analyze-prp-results` command.
- **Update knowledge base** with any new patterns or gotchas discovered.
- **Add new sub-tasks or TODOs** discovered during development to `TASK.md` under a "Discovered During Work" section.
- **Document any deviations** from the original PRP and reasons for changes.

### 📎 Style & Conventions Enhanced
- **Use Python** as the primary language with modern Python 3.9+ features.
- **Follow PEP8**, use type hints, and format with `ruff` (preferred) or `black`.
- **Use `pydantic` for data validation** and leverage v2 features properly.
- Use `FastAPI` for APIs and `SQLAlchemy` (async) for ORM if applicable.
- **Use proper async patterns** - async/await throughout, proper session management.
- Write **docstrings for every function** using the Google style:
  ```python
  def example():
      """
      Brief summary.

      Args:
          param1 (type): Description.

      Returns:
          type: Description.
          
      Raises:
          ValueError: When invalid input provided.
      """
  ```

### 📚 Documentation & Explainability Enhanced
- **Update `README.md`** when new features are added, dependencies change, or setup steps are modified.
- **Comment non-obvious code** and ensure everything is understandable to a mid-level developer.
- **Document architectural decisions** and include rationale for complex implementations.
- When writing complex logic, **add an inline `# Reason:` comment** explaining the why, not just the what.
- **Keep `.env.example` up to date** with all required environment variables and descriptions.

### 🛡️ Security & Best Practices
- **Never hardcode secrets** - always use environment variables.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Femstar08/context-engineering-template](https://github.com/Femstar08/context-engineering-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
