---
trigger: always_on
description: description: llm-food project: Pydantic models for API data validation, serialization, and defining clear data contracts for requests and responses.
---

---
description: llm-food project: Pydantic models for API data validation, serialization, and defining clear data contracts for requests and responses.
globs: llm_food/models.py
alwaysApply: false
---
# Chapter 2: APIDataModels (Pydantic)

In Chapter 1, we explored the [FastAPIServerEndpoints](fastapiserverendpoints.mdc), which define the HTTP API for `llm-food`. We saw how endpoints like `/convert` and `/batch` use `response_model` to specify the structure of their responses. This chapter delves into those Pydantic models, collectively referred to as `APIDataModels`, which are crucial for defining and enforcing data structures for API requests and responses.

## Motivation and Purpose

The primary technical problem `APIDataModels` solve is ensuring **type safety, data validation, and a clear contract** for data exchanged between the client and the server. Without well-defined data models, APIs are prone to errors caused by mismatched data formats, missing fields, or incorrect data types. This can lead to runtime failures, difficult debugging, and a brittle system.

`APIDataModels` in `llm-food` implement the Data Transfer Object (DTO) pattern. They serve as:
-   **A source of truth** for the structure of API payloads.
-   **Automatic validators** for incoming and outgoing data (leveraging FastAPI's Pydantic integration).
-   **Serializers/Deserializers** that convert Python objects to JSON and vice-versa.
-   **Clear documentation** for API consumers about expected data formats.

**Central Use Case:** When a client requests a file conversion via the `POST /convert` endpoint (as discussed in [FastAPIServerEndpoints](fastapiserverendpoints.mdc)), the server processes the file and returns a JSON response. This response must conform to the `ConversionResponse` model. If it does, the client can reliably parse it. If the server attempts to return data in a different structure, FastAPI (using Pydantic) will raise an error, preventing malformed data from being sent.

All these models are defined in `llm_food/models.py`.

## What are Pydantic Models?

Pydantic is a Python library for data validation and settings management using Python type annotations. A Pydantic model is a class that inherits from `pydantic.BaseModel`. You define the fields of your data structure as class attributes with type hints.

```python
# llm_food/models.py (Conceptual Example)
from pydantic import BaseModel
from typing import List

class MySimpleModel(BaseModel):
    name: str
    count: int
    tags: List[str] = [] # Optional field with a default value
```
Pydantic will automatically:
-   Validate that data provided to create an instance of `MySimpleModel` matches the types (e.g., `name` is a string, `count` is an integer).
-   Convert types where possible (e.g., a string `"5"` to an integer `5` for `count`).
-   Provide default values if data for a field is not supplied.
-   Raise validation errors if data is incorrect or missing for required fields.

FastAPI uses these models extensively. When you declare a Pydantic model as a `response_model` for an endpoint, FastAPI ensures the returned data conforms to this model and serializes it to JSON. If used for request bodies, FastAPI validates incoming JSON data against the model.

## Key API Data Models in `llm-food`

Let's examine the core Pydantic models used in `llm-food` for API communication.

### 1. `ConversionResponse`

This model defines the structure of the JSON response for synchronous conversion requests (e.g., `POST /convert` and `GET /convert` endpoints).

**Purpose:** To return the converted text content along with metadata about the original file.

**Structure (`llm_food/models.py`):**
```python
from pydantic import BaseModel
from typing import List

class ConversionResponse(BaseModel):
    filename: str
    content_hash: str
    texts: List[str]
```
-   `filename`: The name of the original file or a derived name from the URL.
-   `content_hash`: A SHA256 hash of the original file content.
-   `texts`: A list of strings, where each string typically represents a page or a section of the converted document (e.g., Markdown content).

**Example JSON Response (for `/convert`):**
```json
{
  "filename": "mydocument.docx",
  "content_hash": "a1b2c3d4e5f6...",
  "texts": ["Converted page 1 content...", "Converted page 2 content..."]
}
```

### 2. Batch Processing Models

Batch processing involves multiple files and asynchronous operations, requiring more complex data models for status updates and results.

#### `FileTaskDetail`

This sub-model provides details about the processing status of an individual file (or a part of it, like a PDF page) within a batch job.

**Purpose:** To give granular status updates for each item in a batch.

**Structure (`llm_food/models.py`):**
```python
from pydantic import BaseModel, Field
from typing import Optional

class FileTaskDetail(BaseModel):
    original_filename: str
    file_type: str
    status: str # e.g., "pending", "processing", "completed", "failed"
    gcs_output_markdown_uri: Optional[str] = None
    error_message: Optional[str] = None

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altaidevorg/llm-food](https://github.com/altaidevorg/llm-food) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
