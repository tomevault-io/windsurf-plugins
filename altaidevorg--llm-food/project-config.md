---
trigger: always_on
description: description: llm-food project: LLMFoodClient, an async Python client for server API interaction, parsing responses into Pydantic models.
---

---
description: llm-food project: LLMFoodClient, an async Python client for server API interaction, parsing responses into Pydantic models.
globs: llm_food/client.py
alwaysApply: false
---
# Chapter 3: LLMFoodClient

In the previous chapter, [APIDataModels (Pydantic)](apidatamodels__pydantic_.mdc), we explored the Pydantic models that define the structure of data exchanged with the `llm-food` server. Now, we will examine the `LLMFoodClient`, an asynchronous Python client library designed to simplify programmatic interaction with the server's API, making direct use of these data models.

## Motivation and Purpose

The `LLMFoodClient` solves the problem of **abstracting away the complexities of direct HTTP communication** with the `llm-food` server. Manually constructing HTTP requests, handling authentication, managing asynchronous operations, parsing JSON responses, and implementing robust error handling can be tedious and error-prone for developers wanting to integrate `llm-food` services into their applications.

The `LLMFoodClient` acts as a **Facade** for the server's API (defined by [FastAPIServerEndpoints](fastapiserverendpoints.mdc)). It provides a clean, high-level, asynchronous interface with methods that directly correspond to the server's API endpoints. Key responsibilities include:
- Asynchronous HTTP request construction and execution using `httpx`.
- Optional API token-based authentication.
- Deserialization of JSON responses into the Pydantic models defined in [APIDataModels (Pydantic)](apidatamodels__pydantic_.mdc) (e.g., `ConversionResponse`, `BatchJobStatusResponse`).
- Standardized error handling by raising a custom `LLMFoodClientError` for API or network issues.
- Providing type-hinted methods for better developer experience and static analysis.

This abstraction significantly simplifies server communication, promotes ease of integration, and serves as the core foundation for the `llm-food` Command Line Interface (CLI).

**Central Use Case:** A developer needs to write a Python script to automatically upload a `.docx` file to the `llm-food` server for conversion to Markdown and then process the returned Markdown content. Instead of using a raw HTTP library, they can use `LLMFoodClient` for a more straightforward and type-safe interaction.

All client logic is encapsulated within `llm_food/client.py`.

## Core Components and Structure

The `LLMFoodClient` is primarily composed of:

1.  **`LLMFoodClient` Class:** The main class providing all client functionalities.
    *   **Initialization (`__init__`)**: Takes the `base_url` of the `llm-food` server and an optional `api_token` for authentication.
2.  **`LLMFoodClientError` Exception:** A custom exception class raised for errors encountered during client operations, such as HTTP errors or request issues. It often includes the HTTP status code and response text from the server for better diagnostics.
3.  **Private `_request` Method:** An internal helper method responsible for:
    *   Constructing the full request URL.
    *   Adding necessary headers (e.g., `Accept: application/json`, `Authorization: Bearer <token>`).
    *   Making the actual asynchronous HTTP request using `httpx.AsyncClient`.
    *   Performing initial response validation (e.g., `response.raise_for_status()`).
    *   Wrapping `httpx` exceptions into `LLMFoodClientError`.
4.  **Public API Methods:** Asynchronous methods that mirror the server endpoints:
    *   `convert_file(file_path: str) -> ConversionResponse`
    *   `convert_url(url_to_convert: str) -> ConversionResponse`
    *   `create_batch_job(file_paths: List[str], output_gcs_path: str) -> Dict[str, Any]`
    *   `get_detailed_batch_job_status(task_id: str) -> BatchJobStatusResponse`
    *   `get_batch_job_results(task_id: str) -> BatchJobOutputResponse`

## How to Use `LLMFoodClient`

Using the `LLMFoodClient` involves instantiating it and then calling its `async` methods.

**1. Initialization:**

```python
# llm_food/client.py
from typing import Optional
# ... other imports

class LLMFoodClient:
    def __init__(self, base_url: str, api_token: Optional[str] = None):
        self.base_url = base_url.rstrip("/")
        self.api_token = api_token
        self.headers = {"Accept": "application/json"}
        if self.api_token:
            self.headers["Authorization"] = f"Bearer {self.api_token}"
```
- `base_url`: The root URL of the `llm-food` server (e.g., `http://localhost:8000`).
- `api_token` (optional): If the server requires authentication, provide the API token here.

**Example Instantiation:**
```python
import asyncio
from llm_food.client import LLMFoodClient, LLMFoodClientError
from llm_food.models import ConversionResponse # From apidatamodels__pydantic_.mdc

async def run_conversion():
    client = LLMFoodClient(
        base_url="http://localhost:8000",
        api_token="your_secret_api_token" # Optional
    )
    # ... use client methods
```

**2. Calling API Methods:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altaidevorg/llm-food](https://github.com/altaidevorg/llm-food) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
