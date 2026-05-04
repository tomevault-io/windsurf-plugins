---
trigger: always_on
description: - ./backend contains the FastAPI backend. It uses uv as package manager.
---

## Gemini Added Memories
- ./backend contains the FastAPI backend. It uses uv as package manager.
- ./frontend contains the TS React frontend. It uses ShadCN components.

## Useful commands:

Run `make` with these targets:
  install                     Install all dependencies for frontend and backend
  dev                         Start the development servers for frontend and backend
  test                        Run tests for frontend and backend
  lint                        Lint the frontend and backend code
  docker-build                Build the production Docker images
  docker-up                   Start the production application with Docker Compose
  docker-down                 Stop the production application
  docker-dev-up               Start the development environment with Docker Compose
  docker-dev-down             Stop the development environment
  clean                       Remove generated files and caches

## Writing Tests

### General Guidance

- When adding tests, first examine existing tests to understand and conform to established conventions.
- Pay close attention to the mocks at the top of existing test files; they reveal critical dependencies and how they are managed in a test environment.

## Git Repo

The main branch for this project is called "master"

## Backend

Of course. Here is a similar markdown for an AI agent on how to write Python code for a backend with FastAPI, SQLAlchemy, and Alembic:

## Backend

When contributing to this Python, FastAPI, and SQLAlchemy codebase, please adhere to the following principles to ensure the code is robust, maintainable, and performs well. The focus is on leveraging modern Python features, functional programming concepts, and the specific strengths of our chosen frameworks.

### Prefer Functional Approaches and Data Classes over Traditional Classes

While Python is a multi-paradigm language that fully supports object-oriented programming, for our backend services, we favor a more functional approach, especially for business logic and data handling.

-   **Simplicity and Predictability**: Functions that operate on data are often simpler to reason about than classes with internal state and methods. This leads to more predictable code with fewer side effects. Pure functions, which always produce the same output for the same input and have no side effects, are the ideal.

-   **Seamless FastAPI Integration**: FastAPI is designed around functions. Dependencies are injected into functions, and route handlers are functions. Writing your logic in functions aligns perfectly with this design, leading to cleaner and more idiomatic FastAPI code.

-   **Data-Oriented Design with Pydantic**: Instead of creating complex classes to hold data, use Pydantic models. Pydantic provides data validation, serialization, and deserialization out of the box, all based on standard Python type hints. This is more declarative and less error-prone than manual implementation.

-   **Reduced Boilerplate**: Traditional classes can introduce boilerplate like `__init__` methods, `self`, and method binding. For many tasks, simple functions operating on Pydantic models or dictionaries are more concise and just as effective.

### Leveraging Python Modules for Encapsulation

Python's module system is the primary way to organize and encapsulate code. We prefer using modules to control visibility over class-based access modifiers like `_` or `__`.

-   **Clear Public API**: Anything you import from a module is part of its public API. Anything you don't is considered private. This is a simple and effective way to define module boundaries.

-   **Enhanced Testability**: Test the public functions and interfaces of your modules. If you find yourself needing to test an "internal" function, consider if it should be part of the public API or if the module should be broken down further.

-   **Reduced Coupling**: Well-defined modules with clear public APIs reduce coupling between different parts of the application, making it easier to refactor and maintain.

### Static Typing with Pydantic and Type Hints

Python's optional static typing is a powerful tool for writing robust and maintainable code. We use it extensively.

-   **Avoid `Any`**: The `Any` type subverts the type checker. Avoid it whenever possible. If you have a truly unknown type, be explicit about how you handle it.

-   **Leverage Pydantic for Validation**: Use Pydantic models for all data coming into and out of your API. This includes request bodies, query parameters, and response models. This ensures that your data is always in the expected shape.

-   **Use Type Hints Everywhere**: All function signatures should have type hints. This improves readability and allows static analysis tools to catch errors before they happen in production.

### Embracing Python's Built-in Data Structures and Comprehensions

Python has a rich set of built-in data structures and powerful syntax for working with them.

-   **List Comprehensions and Generator Expressions**: Prefer list comprehensions and generator expressions over `for` loops for creating lists and other collections. They are more concise and often more performant.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeonMusCoden/LetterFeed](https://github.com/LeonMusCoden/LetterFeed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
