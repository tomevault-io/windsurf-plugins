---
trigger: always_on
description: description: llm-food project: SynchronousConversionService for on-demand single document/URL to Markdown conversion.
---

---
description: llm-food project: SynchronousConversionService for on-demand single document/URL to Markdown conversion.
globs: 
alwaysApply: false
---
# Chapter 4: SynchronousConversionService

In the [LLMFoodClient](llmfoodclient.mdc) chapter, we saw how a client application can make requests to the `llm-food` server for document conversion. This chapter dives into the server-side component responsible for handling these immediate, single-file conversion requests: the `SynchronousConversionService`.

## Motivation and Purpose

The `SynchronousConversionService` addresses the need for a centralized, reusable, and testable component that performs on-demand conversion of single documents or URL content to Markdown. When a client sends a file or URL to the `/convert` endpoints (detailed in [FastAPIServerEndpoints](fastapiserverendpoints.mdc)) and expects an immediate Markdown response, this service's logic is invoked.

Its key characteristics include:
- **Synchronous Interaction Model:** Clients make a request and block (wait) until the conversion is complete and the Markdown result is returned. This is suitable for interactive use cases or scenarios where immediate processing is required.
- **Dynamic Processor Selection:** It intelligently chooses the correct parsing/conversion library based on the input file's extension (e.g., PDF, DOCX, HTML).
- **Configurable PDF Strategy:** For PDF documents, it employs a specific [PDFProcessingStrategy (Synchronous)](pdfprocessingstrategy__synchronous_.mdc), which can be configured (e.g., Gemini, pymupdf4llm, pypdf2).
- **File Size Validation:** It incorporates logic to check file sizes against configured limits.
- **Centralized Logic:** It encapsulates the core single-file transformation logic, making the API endpoints cleaner and the conversion process itself easier to manage and test.

This service contrasts with the [BatchJobOrchestrator](batchjoborchestrator.mdc), which is designed for asynchronous processing of multiple files.

**Central Use Case:** A user uploads a `.docx` file via a web interface that calls the `POST /convert` API endpoint. The `FastAPIServerEndpoints` layer receives the request, validates the file size, and then invokes the `SynchronousConversionService` logic with the file content. The service identifies it as a DOCX file, uses the `mammoth` library to convert it to HTML, then `markdownify` to convert HTML to Markdown, and returns the Markdown text. The client receives this Markdown in the HTTP response.

While not a standalone class in the current codebase, the logic for the `SynchronousConversionService` is primarily encapsulated within the `_process_file_content` function and its helper processing functions (e.g., `_process_docx_sync`, `_process_pdf_pymupdf4llm_sync`) found in `llm_food/app.py`.

## Key Responsibilities and Mechanisms

### 1. Content Handling and Initial Processing
The service logic is typically invoked after the [FastAPIServerEndpoints](fastapiserverendpoints.mdc) layer has received an uploaded file or fetched content from a URL.

For file uploads (`POST /convert`):
```python
# llm_food/app.py (Simplified from convert_file_upload)
# async def convert_file_upload(file: UploadFile = File(...)):
#     ext = os.path.splitext(file.filename)[1].lower()
#     content = await file.read() # Read file content into bytes

#     # File size validation (see below)
#     # ...

#     pdf_backend_choice = get_pdf_backend() # Get configured PDF strategy

#     # Invoke the core service logic
#     texts_list = await _process_file_content(ext, content, pdf_backend_choice)

#     # Return ConversionResponse (see APIDataModels (Pydantic))
#     # ...
```
- The endpoint reads the file content into `bytes`.
- It determines the file extension and configured PDF backend.
- It then calls `_process_file_content`, which embodies the core of the `SynchronousConversionService`.

For URL conversions (`GET /convert`):
```python
# llm_food/app.py (Simplified from convert_url)
# async def convert_url(url: str = Query(...)):
#     # ... (fetch URL content into content_bytes using httpx) ...
#     # content_bytes = html_content.encode("utf-8")

#     # For HTML, trafilatura is used directly in the endpoint handler
#     # or _process_file_content could be extended for URLs.
#     extracted_text = trafilatura.extract(html_content, output_format="markdown")
#     texts_list = [extracted_text if extracted_text is not None else ""]
#     # ... return ConversionResponse ...
```
- The `/convert` GET endpoint currently handles HTML URL conversion directly using `trafilatura`. The `_process_file_content` function also includes logic for HTML files if they were uploaded.

### 2. File Size Validation
Before processing, file sizes are validated (primarily for uploads) to prevent resource exhaustion. This check occurs in the API endpoint handler before delegating to the core conversion logic.

```python
# llm_food/app.py (within convert_file_upload)
max_size = get_max_file_size_bytes() # From config
if max_size is not None and len(content) > max_size:
    raise HTTPException(
        status_code=413,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altaidevorg/llm-food](https://github.com/altaidevorg/llm-food) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
