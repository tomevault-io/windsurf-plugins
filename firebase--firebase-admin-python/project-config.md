---
trigger: always_on
description: This document provides AI agents with a comprehensive guide to the conventions, design patterns, and architectural nuances of the Firebase Admin Python SDK. Adhering to this guide ensures that all contributions are idiomatic and align with the existing codebase.
---

# Firebase Admin Python SDK - Agent Guide

This document provides AI agents with a comprehensive guide to the conventions, design patterns, and architectural nuances of the Firebase Admin Python SDK. Adhering to this guide ensures that all contributions are idiomatic and align with the existing codebase.

## 1. High-Level Overview

The Firebase Admin Python SDK provides a Pythonic interface to Firebase services. Its design emphasizes thread-safety, a consistent and predictable API, and seamless integration with Google Cloud Platform services.

## 2. Directory Structure

-   `firebase_admin/`: The main package directory.
    -   `__init__.py`: The primary entry point. It exposes the `initialize_app()` function and manages the lifecycle of `App` instances.
    -   `exceptions.py`: Defines the custom exception hierarchy for the SDK.
    -   `_http_client.py`: Contains the centralized `JsonHttpClient` and `HttpxAsyncClient` for all outgoing HTTP requests.
    -   Service modules (e.g., `auth.py`, `db.py`, `messaging.py`): Each module contains the logic for a specific Firebase service.
-   `tests/`: Contains all unit tests.
    -   `tests/resources/`: Contains mock data, keys, and other test assets.
-   `integration/`: Contains all integration tests.* 
    -   These integration tests require a real Firebase project to run against.
    -   `integration/conftest.py`: Contains provides configurations for these integration tests including how credentials are provided through pytest.
-   `snippets/`: Contains code snippets used in documentation.
-   `setup.py`: Package definition, including the required environment dependencies.
-   `requirements.txt`: A list of all development dependencies.
-   `.pylintrc`: Configuration file for the `pylint` linter.
-   `CONTRIBUTING.md`: General guidelines for human contributors. Your instructions here supersede this file.

## 3. Core Design Patterns

### Initialization

The SDK is initialized by calling the `initialize_app(credential, options)` function. This creates a default `App` instance that SDK modules use implicitly. For multi-project use cases, named apps can be created by providing a `name` argument: `initialize_app(credential, options, name='my_app')`.

### Service Clients

Service clients are accessed via module-level factory functions. These functions automatically use the default app unless a specific `App` object is provided via the `app` parameter. The clients are created lazily and cached for the lifetime of the application.

- **Direct Action Modules (auth, db)**: Some modules provide functions that perform actions directly.
- **Client Factory Modules (firestore, storage)**: Other modules have a function (e.g., client() or bucket()) that returns a client object, which you then use for operations.


### Error Handling

-   All SDK-specific exceptions inherit from `firebase_admin.exceptions.FirebaseError`.
-   Specific error conditions are represented by subclasses, such as `firebase_admin.exceptions.InvalidArgumentError` and `firebase_admin.exceptions.UnauthenticatedError`.
-   Each service may additionaly define exceptions under these subclasses and apply them by passing a handle function to `_utils.handle_platform_error_from_requests()` or `_utils.handle_platform_error_from_httpx()`. Each services error handling patterns should be considered before making changes.

### HTTP Communication

-   All synchronous HTTP requests are made through the `JsonHttpClient` class in `firebase_admin._http_client`.
-   All asynchronous HTTP requests are made through the `HttpxAsyncClient` class in `firebase_admin._http_client`.
-   These clients handle authentication and retries for all API calls.

### Asynchronous Operations

Asynchronous operations are supported using Python's `asyncio` library. Asynchronous methods are typically named with an `_async` suffix (e.g., `messaging.send_each_async()`).

## 4. Coding Style and Naming Conventions

-   **Formatting:** This project uses **pylint** to enforce code style and detect potential errors. Before submitting code, you **must** run the linter and ensure your changes do not introduce any new errors. Run the linter from the repository's root directory with the following command:
    ```bash
    ./lint.sh all   # Lint all source files
    ```
    or 
    ```bash
    ./lint.sh   # Lint locally modified source files
    ```
-   **Naming:**
    -   Classes: `PascalCase` (e.g., `FirebaseError`).
    -   Methods and Functions: `snake_case` (e.g., `initialize_app`).
    -   Private Members: An underscore prefix (e.g., `_http_client`).
    -   Constants: `UPPER_SNAKE_CASE` (e.g., `INVALID_ARGUMENT`).

## 5. Testing Philosophy

-   **Unit Tests:**
    -   Located in the `tests/` directory.
    -   Test files follow the `test_*.py` naming convention.
    -   Unit tests can be run using the following command:
        ```bash
        pytest
        ```
-   **Integration Tests:**
    -   Located in the `integration/` directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firebase/firebase-admin-python](https://github.com/firebase/firebase-admin-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
