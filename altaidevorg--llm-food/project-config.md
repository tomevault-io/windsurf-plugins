---
trigger: always_on
description: description: llm-food project: PDFProcessingStrategy (Synchronous) for configurable PDF-to-Markdown conversion in synchronous API calls via /convert endpoint.
---

---
description: llm-food project: PDFProcessingStrategy (Synchronous) for configurable PDF-to-Markdown conversion in synchronous API calls via /convert endpoint.
globs: 
alwaysApply: false
---
# Chapter 6: PDFProcessingStrategy (Synchronous)

Welcome to Chapter 6! In the previous chapter, [BatchJobOrchestrator](batchjoborchestrator.mdc), we explored how `llm-food` handles asynchronous batch processing of multiple files, including PDFs via the Gemini Batch API. This chapter shifts focus back to synchronous, single-file processing, specifically detailing the **PDFProcessingStrategy (Synchronous)** employed by the `/convert` endpoint.

## Motivation and Purpose

Processing PDF documents can be complex due to their varied nature (text-based, image-based, mixed) and the desired output quality (simple text extraction vs. layout-preserving Markdown). Different PDF processing libraries and services offer distinct advantages in terms of output quality, speed, licensing, and cost (e.g., using cloud AI services).

The `PDFProcessingStrategy (Synchronous)` addresses the technical problem of **providing flexibility in how PDF files are converted to Markdown within the synchronous `/convert` API endpoint**. Instead of hardcoding a single PDF processing method, `llm-food` implements the Strategy design pattern. This allows the server's PDF processing behavior to be configured via the `PDF_BACKEND` environment variable, enabling users or administrators to choose a backend (e.g., Google Gemini, `pymupdf4llm`, `pypdf2`) that best suits their requirements without altering the core API endpoint logic.

This pattern promotes:
- **Flexibility**: Adapt to different PDF types and quality needs.
- **Extensibility**: Easily add new PDF processing methods in the future.
- **Maintainability**: Decouples PDF processing choices from the main request handling logic.

**Central Use Case:** A user uploads a PDF file (e.g., `report.pdf`) to the `POST /convert` endpoint. The `llm-food` server, based on its `PDF_BACKEND` configuration (e.g., set to `"pymupdf4llm"`), dynamically selects and uses the `pymupdf4llm` library to convert the PDF content to Markdown. The resulting Markdown is then returned to the user in the API response. If `PDF_BACKEND` were set to `"gemini"`, the server would instead use the Google Gemini API for OCR and Markdown conversion of that same PDF.

This strategy is distinct from the batch PDF processing discussed in [BatchJobOrchestrator](batchjoborchestrator.mdc), which *exclusively* uses the Gemini Batch API for scalability.

## How It Works: Configuration and Dispatch

The core of this strategy lies in the interaction between server configuration and the dispatch logic within the `_process_file_content` function (introduced in [SynchronousConversionService](synchronousconversionservice.mdc) and located in `llm_food/app.py`).

### 1. Configuration via `PDF_BACKEND`

The choice of PDF processing backend is determined by the `PDF_BACKEND` environment variable. The `llm_food/config.py` file provides a function to access this value:

```python
# llm_food/config.py
import os

def get_pdf_backend():
    return os.getenv("PDF_BACKEND", "gemini") # Defaults to "gemini"
```
- This function retrieves the value of `PDF_BACKEND`. If the variable is not set, it defaults to `"gemini"`.
- Supported values typically include `"gemini"`, `"pymupdf4llm"`, and `"pypdf2"`.

### 2. Dynamic Dispatch in `_process_file_content`

The `_process_file_content` function in `llm_food/app.py` acts as the "Context" in the Strategy pattern. When it encounters a PDF file (extension `.pdf`), it uses the `pdf_backend_choice` (obtained from `get_pdf_backend()`) to select and execute the appropriate PDF processing logic.

```python
# llm_food/app.py (Simplified excerpt from _process_file_content)
async def _process_file_content(
    ext: str, content: bytes, pdf_backend_choice: str
) -> List[str]:
    texts_list: List[str] = []
    if ext == ".pdf":
        if pdf_backend_choice == "pymupdf4llm":
            texts_list = await asyncio.to_thread(_process_pdf_pymupdf4llm_sync, content)
        elif pdf_backend_choice == "pypdf2":
            texts_list = await asyncio.to_thread(_process_pdf_pypdf2_sync, content)
        elif pdf_backend_choice == "gemini":
            # ... Gemini single PDF processing logic ...
            # (Details covered in the next section)
            pass # Placeholder for Gemini logic
        else:
            texts_list = ["Invalid PDF backend specified."]
    # ... (elif blocks for other file types like .docx, .pptx) ...
    return texts_list
```
- **Input**: `ext` (file extension), `content` (file bytes), `pdf_backend_choice` (the string from `PDF_BACKEND`).
- **Behavior**:
  - If `ext` is `".pdf"`, it enters the PDF processing block.
  - An `if/elif/else` structure checks `pdf_backend_choice`.
  - Based on the choice, it calls a specific helper function (e.g., `_process_pdf_pymupdf4llm_sync`) or executes inline logic (for Gemini).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altaidevorg/llm-food](https://github.com/altaidevorg/llm-food) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
