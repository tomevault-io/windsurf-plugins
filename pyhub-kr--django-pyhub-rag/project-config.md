---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

django-pyhub-rag is a Django library that enables easy RAG (Retrieval Augmented Generation) implementation in Django projects. It supports Windows/Mac/Linux and provides PDF parsing with image extraction, vector store integration, and multiple LLM provider support.

## Development Commands

### Common Development Tasks

```bash
# Run tests
make test

# Run specific test file
make test tests/test_llm.py

# Format code (auto-fix)
make format

# Check code formatting (no changes)
make lint

# Build package
make build

# Run documentation server locally
make docs

# Build documentation
make docs-build
```

### CLI Commands

```bash
# Parse PDF with Upstage API (extracts images and generates descriptions)
pyhub.parser upstage file.pdf

# LLM operations
pyhub.llm ask "your question"
pyhub.llm describe image.png
pyhub.llm embed "text to embed"
pyhub.llm chat  # Interactive chat session
pyhub.llm compare "question" --models gpt-4o-mini --models claude-3-haiku
pyhub.llm agent run "Calculate 25 * 4"  # Run React Agent
pyhub.llm agent list-tools  # List available tools

# RAG operations
pyhub.rag sqlite-vec create-table  # Create vector table
pyhub.rag sqlite-vec load-jsonl data.jsonl  # Load JSONL data

# Document management
pyhub.doku  # Document management commands

# Run web server
pyhub.web
```

## Development Notes

- 테스트 시에는 pytest가 아니라 make test 명령을 활용한다.

## Testing

### Running Tests

#### Command Line (Recommended)
```bash
# Run all tests
make test

# Run specific test file  
make test tests/test_llm.py

# Using pytest directly
python -m pytest tests/test_fields_core.py -v

# Skip vector database tests (when sqlite-vec not available)
SKIP_DATABASE_TESTS=1 python -m pytest tests/
```

#### PyCharm/IDE Setup
For PyCharm or other IDEs using Django's test runner, use the dedicated settings module:

**Settings → Languages & Frameworks → Django → Settings:**
- Django project root: `/path/to/django-pyhub-rag`
- Settings: `tests.django_settings`
- Manage script: `/path/to/django-pyhub-rag/tests/manage.py`

**Run Configuration:**
- Test: Django tests
- Settings: `tests.django_settings`
- Working directory: `/path/to/django-pyhub-rag`

This configuration excludes vector-related apps to avoid sqlite-vec dependencies in the test environment.

### Test Organization
- All tests are centralized in the `tests/` directory
- Core field tests: `tests/test_fields_core.py`
- LLM file utilities: `tests/test_llm_files.py`
- Cache functionality: `tests/test_caches.py`
- Vector database models: `tests/test_models.py` (requires vector extensions)

## Architecture

### Django Apps Structure

#### 1. **core** - Base Utilities
- **Purpose**: Provides base utilities and custom fields used across the project
- **Key Features**:
  - Custom model fields (`PDFFileField`, `PageNumbersField`)
  - Static file management (htmx, alpine.js, marked.js, etc.)
  - Base templates and error pages
  - Configuration file management command (`pyhub_toml`)

#### 2. **doku** - Document Management
- **Purpose**: Handles PDF document upload, parsing, vectorization, and search
- **Key Models**:
  - `Document`: Manages uploaded PDF documents
  - `DocumentParseJob`: Tracks document parsing jobs
  - `VectorDocument`: Stores vectorized document chunks
  - `VectorDocumentImage`: Stores extracted images with AI-generated descriptions
- **Workflow**:
  1. PDF upload → Document creation
  2. Automatic DocumentParseJob creation (Django Lifecycle)
  3. Document parsing via Upstage API
  4. Text/image extraction and VectorDocument creation
  5. Automatic embedding generation and storage

#### 3. **llm** - LLM Integration
- **Purpose**: Provides unified interface for various LLM providers
- **Supported Providers**:
  - OpenAI (GPT-3.5, GPT-4)
  - Anthropic (Claude)
  - Google (Gemini)
  - Ollama (local models)
  - Upstage (Solar)
- **Key Features**:
  - Unified API (`LLM.create()`)
  - Async/streaming response support
  - Structured responses (choices parameter with JSON Schema)
  - Multimodal support (image input)
  - Automatic caching mechanism
  - React Agent system with tool integration
  - Input validation for tools to reduce unnecessary API calls

#### 4. **parser** - Document Parsing
- **Purpose**: PDF document parsing and image extraction
- **Key Features**:
  - Upstage Document Parse API integration
  - Image/table extraction and description generation
  - Template-based prompt management
  - CLI command: `pyhub.parser upstage`
- **Parsing Results**:
  - Separated text, images, and tables
  - Page-by-page structured data
  - AI-generated image/table descriptions

#### 5. **rag** - Vector Store
- **Purpose**: Vector database abstraction and similarity search
- **Supported Databases**:
  - PostgreSQL (pgvector extension)
  - SQLite (sqlite-vec extension)
- **Key Features**:
  - DB-agnostic VectorField implementation
  - Cosine similarity-based search
  - Langchain-compatible interface
  - Automatic embedding generation
  - Async search support

#### 6. **ui** - UI Components
- **Purpose**: Provides reusable UI components
- **Key Components**:
  - Modal: Modal dialogs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyhub-kr/django-pyhub-rag](https://github.com/pyhub-kr/django-pyhub-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
