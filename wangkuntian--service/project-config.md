---
trigger: always_on
description: description: Comprehensive guidelines for Python development, covering code organization, performance, security, testing, and more.  These rules promote maintainable, efficient, and secure Python codebases.
---

---
description: Comprehensive guidelines for Python development, covering code organization, performance, security, testing, and more.  These rules promote maintainable, efficient, and secure Python codebases.
globs: *.py
---
# Python Best Practices and Coding Standards

This document outlines comprehensive best practices and coding standards for Python development, aiming to promote clean, efficient, maintainable, and secure code.

## 1. Code Organization and Structure

### 1.1. Directory Structure Best Practices

*   **Flat is better than nested (but not always).**  Start with a simple structure and refactor as needed.
*   **Packages vs. Modules:** Use packages (directories with `__init__.py`) for logical grouping of modules.
*   **src layout:** Consider using a `src` directory to separate application code from project-level files (setup.py, requirements.txt, etc.). This helps avoid import conflicts and clarifies the project's boundaries.
*   **Typical Project Structure:**

    
    project_name/
    ├── src/
    │   ├── package_name/
    │   │   ├── __init__.py
    │   │   ├── module1.py
    │   │   ├── module2.py
    │   ├── main.py  # Entry point
    ├── tests/
    │   ├── __init__.py
    │   ├── test_module1.py
    │   ├── test_module2.py
    ├── docs/
    │   ├── conf.py
    │   ├── index.rst
    ├── .gitignore
    ├── pyproject.toml or setup.py
    ├── README.md
    ├── requirements.txt or requirements-dev.txt
    

### 1.2. File Naming Conventions

*   **Modules:**  Lowercase, with underscores for readability (e.g., `my_module.py`).
*   **Packages:** Lowercase (e.g., `my_package`). Avoid underscores unless necessary.
*   **Tests:** Start with `test_` (e.g., `test_my_module.py`).

### 1.3. Module Organization Best Practices

*   **Single Responsibility Principle:** Each module should have a well-defined purpose.
*   **Imports:**
    *   Order: standard library, third-party, local.
    *   Absolute imports are generally preferred (e.g., `from my_package.module1 import function1`).
    *   Use explicit relative imports (`from . import sibling_module`) when dealing with complex package layouts where absolute imports are overly verbose or impractical.
*   **Constants:**  Define module-level constants in uppercase (e.g., `MAX_ITERATIONS = 100`).
*   **Dunder names:** `__all__`, `__version__`, etc. should be after the module docstring but before any imports (except `from __future__`).  Use `__all__` to explicitly define the public API.

### 1.4. Component Architecture Recommendations

*   **Layered Architecture:** Suitable for larger applications, separating concerns into presentation, business logic, and data access layers.
*   **Microservices:**  For very large applications, consider breaking the system into smaller, independent services.
*   **Hexagonal/Clean Architecture:** Emphasizes decoupling business logic from external dependencies like databases and frameworks.
*   **Dependency Injection:** Use dependency injection to improve testability and reduce coupling.

### 1.5. Code Splitting Strategies

*   **By Functionality:**  Split code into modules based on distinct functionalities (e.g., user management, data processing).
*   **By Layer:** Separate presentation, business logic, and data access code.
*   **Lazy Loading:** Use `importlib.import_module()` to load modules on demand, improving startup time.
*   **Conditional Imports:** Import modules only when needed, based on certain conditions.

## 2. Common Patterns and Anti-patterns

### 2.1. Design Patterns

*   **Singleton:**  Restrict instantiation of a class to one object.
*   **Factory:**  Create objects without specifying the exact class to be created.
*   **Observer:**  Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified.
*   **Strategy:**  Define a family of algorithms, encapsulate each one, and make them interchangeable.
*   **Decorator:**  Add responsibilities to objects dynamically.
*   **Context Manager:** Guarantees resources are properly cleaned up (e.g., files are closed).

### 2.2. Recommended Approaches for Common Tasks

*   **Data Validation:** Use libraries like `pydantic` or `marshmallow` for data validation and serialization.
*   **Configuration Management:** Use libraries like `python-decouple`, `dynaconf` or standard library's `configparser` to manage environment-specific settings.
*   **Logging:** Use the `logging` module for structured logging. Configure log levels and handlers appropriately.
*   **Command-Line Interfaces:** Use `argparse`, `click` or `typer` for creating command-line interfaces.
*   **Asynchronous Programming:** Use `asyncio` for I/O-bound and concurrency problems.

### 2.3. Anti-patterns and Code Smells

*   **God Class:** A class that does too much.  Break it down into smaller, more focused classes.
*   **Shotgun Surgery:**  Making small changes to many different classes at once. Indicates poor cohesion.
*   **Spaghetti Code:**  Unstructured and difficult-to-follow code.  Refactor using well-defined functions and classes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangkuntian/service](https://github.com/wangkuntian/service) — distributed by [TomeVault](https://tomevault.io/claim/wangkuntian).
<!-- tomevault:4.0:windsurf_rules:2026-04-18 -->
