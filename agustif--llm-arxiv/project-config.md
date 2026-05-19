---
trigger: always_on
description: This document provides guidance for AI agents assisting with the development and maintenance of the `llm-arxiv` repository.
---

# AI Agent Instructions for `llm-arxiv`

This document provides guidance for AI agents assisting with the development and maintenance of the `llm-arxiv` repository.

## Project Overview

`llm-arxiv` is a Python-based plugin for the [LLM CLI tool](https://llm.datasette.io/) that enables users to load and process academic papers from arXiv. It fetches paper metadata and content, typically the PDF, and makes it available for language model processing.

## Key Files and Directories

*   **`llm_arxiv.py`**: This is the core file containing the plugin's logic. It implements the LLM plugin interface and handles fetching and processing arXiv papers. The `[project.entry-points.llm]` section in `pyproject.toml` points to this file (`arxiv = "llm_arxiv"`).
*   **`pyproject.toml`**: The main configuration file for the project. It defines dependencies (e.g., `llm`, `arxiv`, `PyMuPDF`), build system settings, project metadata, and the plugin entry point.
*   **`arxiv.py`**: This file might contain utility functions or a script related to interacting with the `arxiv` package or API. Be cautious if modifying, as its role needs to be clearly understood in context of the main `llm_arxiv.py` plugin.
*   **`fitz.py`**: This file likely contains utility functions or a script related to PDF processing, using `PyMuPDF` (which provides Fitz bindings). Similar to `arxiv.py`, understand its specific role before making changes.
*   **`README.md`**: The primary documentation for human users of the plugin. It should be kept up-to-date with features, installation instructions, and usage examples.
*   **`AGENTS.md`**: (This file) Contains specific instructions and context for AI agents working on this codebase.
*   **`tests/`**: This directory contains automated tests for the plugin, likely using `pytest`. New features and bug fixes should ideally be accompanied by relevant tests.
*   **`.github/workflows/`**: Contains GitHub Actions workflow definitions, for example, for running tests automatically.

## Common Development Tasks

When asked to perform tasks, consider the following:

*   **Adding Features**:
    *   Modifications will likely center around `llm_arxiv.py`.
    *   Consider how new features impact dependencies (`pyproject.toml`).
    *   Add corresponding tests in the `tests/` directory.
    *   Update `README.md` with user-facing documentation for the new feature.
*   **Bug Fixing**:
    *   Identify the relevant module (`llm_arxiv.py`, `arxiv.py`, `fitz.py`).
    *   Write a test case that reproduces the bug if possible.
    *   Ensure the fix doesn't break existing functionality by running all tests.
*   **Dependency Management**:
    *   Changes to dependencies are made in `pyproject.toml`.
    *   Be mindful of version compatibility.
*   **Documentation**:
    *   Keep `README.md` clear and up-to-date.
    *   Update this file (`AGENTS.md`) if there are significant changes to the development workflow or codebase structure relevant to AI agents.

## Important Considerations

*   **arXiv API Usage**: If directly interacting with the arXiv API (via the `arxiv` package or otherwise), be mindful of rate limits and terms of service.
*   **PDF Parsing**: PDF parsing can be complex. `PyMuPDF` (Fitz) is used. Ensure robustness and handle potential errors gracefully.
*   **Code Style and Quality**: Follow existing code style. Ensure code is clear, well-commented where non-obvious, and efficient.
*   **Testing**: Always aim to maintain or increase test coverage.

By following these guidelines, AI agents can contribute effectively to the `llm-arxiv` project. 

## Known Issues and Debugging Notes

### Image Resizing Bug (Incorrectly Resizes to 1x1 Pixels)

*   **Problem Description**:
    *   When using the `llm arxiv ... -r` command (enabling image resizing), all extracted images from the PDF are being incorrectly resized to 1x1 pixels.
    *   This causes any downstream vision-enabled LLM to interpret these images as simple solid blocks of color, rather than recognizing their actual content.
    *   The issue is specific to the resizing operation. If resizing is disabled (e.g., `llm arxiv ID -i all` without `-r`, or via the fragment loader `llm -f arxiv:ID?i=all`), images are processed and described correctly by the LLM (though at their original, unresized dimensions).

*   **Location of Buggy Code**:
    *   File: `llm_arxiv.py`
    *   Function: `_process_arxiv_paper`
    *   Specific Block: The section responsible for calculating `new_width` and `new_height` within the `if perform_resize:` block. Debug logs confirm that `new_width` and `new_height` are both evaluating to `1` before being passed to `img.resize()`.

*   **Debugging Steps (to isolate the miscalculation)**:
    1.  Focus on the arithmetic that calculates `new_width` and `new_height`:
        ```python
        if img.width > img.height:
            new_width = max_dim_to_use 
            new_height = max(1, int(max_dim_to_use * img.height / img.width))
        else:
            new_height = max_dim_to_use
            new_width = max(1, int(max_dim_to_use * img.width / img.height))
        ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agustif/llm-arxiv](https://github.com/agustif/llm-arxiv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
