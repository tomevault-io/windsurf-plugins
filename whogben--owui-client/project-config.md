---
trigger: always_on
description: This project aims to create a robust async Python client for the Open WebUI API that mirrors the backend structure of the main Open WebUI project.
---

# Entire Project Instructions

This project aims to create a robust async Python client for the Open WebUI API that mirrors the backend structure of the main Open WebUI project.

## References
Access these references when looking up details of Open WebUI.

### API Reference
Located at `owui_client/refs/owui_openapi_main.json` is the full Open WebUI API spec.
Use this to lookup the descriptions of specific endpoints.
Unfortunately, there is no details on the possible parameters or returns in the Open API spec,
so it is incomplete, so you must look up the details in the backend source code to know
what parameters they accept, and what response fields they may return.

### Open WebUI Backend Source Code
Located in `owui_client/refs/owui_source_main/backend/open_webui` is the full Open WebUI backend source code.
Within that directory focus on:
- `/main.py` top level routers and entry point for the Open WebUI FastAPI app.
- `/routers/..` lower level routers describing specific endpoints/
- `/models/..` additional information and models providing details on the parameters and fields.

Always inspect the backend and locate both the models and the endpoint definitions before
working on related content - this ensures that your information is up to date, as Open WebUI changes often.

### Open WebUI Frontend Source Code
Located in `owui_client/refs/owui_source_main/src` is the full Open WebUI frontend source code.
While we are not directly replicating anything from the frontend, it is often necesary to examine how the frontend uses a given model or endpoint to learn what parameter values are valid. Utilize the frontend code when you are researching to document a model or endpoint.
- `/routes/..`
- `/lib/apis..`
- `/lib/components` 

## VENV AND ENV
There is a .env file at the root of the project - you can't read it for security, but trust it is there!
There is also a .venv at the root of the project: Always use this .venv, never use system python.

## Architecture & Organization Rules

1.  **Mirror Backend Structure**:
    - The client structure must exactly match the Open WebUI backend source structure located in `owui_client/refs/owui_source_main/backend/open_webui`.
    - **Models**: If a Pydantic model is defined in `backend/open_webui/models/auths.py`, its client counterpart must reside in `owui_client/models/auths.py`.
    - **Routers/Resources**: If an endpoint is defined in `backend/open_webui/routers/auths.py`, the client resource class (e.g., `AuthsClient`) must reside in `owui_client/routers/auths.py`.

2.  **Client Assembly**:
    - `owui_client/client.py`: This is the main entry point. It defines the `OpenWebUI` class, which inherits from `OWUIClientBase`.
    - This main class is composed of sub-clients (Resources) that represent each router.
    - Example: `self.auths = AuthsClient(self)` inside `OpenWebUI.__init__`.

3.  **Base Classes**:
    - `OWUIClientBase` (in `owui_client/client_base.py`): Handles the low-level `_request` logic, authentication, and error parsing.
    - `ResourceBase` (in `owui_client/client_base.py`): The parent class for all router resources (e.g., `AuthsClient`). It delegates requests back to the main client.

4.  **Strict Naming**:
    - Maintain the same file names and roughly the same class/variable names as the backend to ensure easy navigation and updates.

5.  **Version Management**:
    - The version number is defined in `owui_client/pyproject.toml`.
    - It is also available at runtime via `owui_client.__version__` (defined in `owui_client/src/owui_client/__init__.py`).
    - **CRITICAL**: These two versions must ALWAYS be kept in sync. When bumping the version in `pyproject.toml`, you must also update `__init__.py`.
    - Versioning is compatibility-driven and follows the Open WebUI version we built/tested against:
        - `client.major` tracks `openwebui.minor`
        - `client.minor` tracks `openwebui.patch`
        - `client.patch` tracks client-only fixes/improvements made while targeting that same Open WebUI version
    - Example mapping:
        - Open WebUI `0.8.12` => client baseline `8.12.0`
        - A subsequent client fix while still targeting Open WebUI `0.8.12` => `8.12.1`
    - Do not auto-bump major/minor independently of the targeted Open WebUI version. Major/minor should change only when the targeted Open WebUI minor/patch changes.
    - When changing package version and/or target Open WebUI version, update `README.md` so the `Target Open WebUI Version` section explicitly states the currently targeted Open WebUI version and matching client version example.
    - When changing package version for a release, add a dated `## [version] - YYYY-MM-DD` section to `CHANGELOG.md` (see **Changelog** below).

### Changelog

- **Source of truth**: `CHANGELOG.md` at the repository root, in the spirit of [Keep a Changelog](https://keepachangelog.com/) (`[Unreleased]` plus one `## [version] - date` section per release).
- **Categories**: Use **Added**, **Changed**, **Fixed**, **Removed**, and **Deprecated** when they apply; skip empty sections.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whogben/owui_client](https://github.com/whogben/owui_client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
