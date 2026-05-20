---
trigger: always_on
description: This document captures best practices and essential information for working with the Archive Agent codebase using Claude Code.
---

# Claude Code Development Guide for Archive Agent

This document captures best practices and essential information for working with the Archive Agent codebase using Claude Code.

## Overview

Archive Agent is an intelligent file indexer with powerful AI search (RAG engine), automatic OCR, and a seamless MCP interface. It processes documents, chunks them semantically, stores embeddings in Qdrant, and provides semantic search capabilities.

## Key Architecture Components

### Core Processing Pipeline
1. **File Tracking**: Pattern-based file selection and change detection
2. **File Processing**: Conversion to text with OCR for images/PDFs  
3. **Semantic Chunking**: AI-powered text chunking with context headers
4. **Embedding**: Vector embeddings stored in Qdrant database
5. **RAG Query**: Retrieval, reranking, expansion, and answer generation

### Important Modules
- `archive_agent/data/FileData.py` - File processing and payload creation
- `archive_agent/core/ProgressManager.py` - Centralized hierarchical progress management
- `archive_agent/data/loader/pdf.py` - PDF processing with business logic (OCR strategy resolution)
- `archive_agent/data/loader/PdfDocument.py` - Clean PDF abstraction interfaces (PdfDocument, PdfPage, PdfPageContent)
- `archive_agent/data/loader/backend/pdf_pymupdf.py` - PyMuPDF implementation backend (fully isolated)
- `archive_agent/data/processor/VisionProcessor.py` - Parallel vision processing
- `archive_agent/data/processor/EmbedProcessor.py` - Parallel chunk embedding
- `archive_agent/db/QdrantManager.py` - Vector database operations  
- `archive_agent/db/QdrantSchema.py` - Qdrant payload schema (Pydantic models)
- `archive_agent/core/ContextManager.py` - Application context initialization
- `archive_agent/core/CommitManager.py` - Commit orchestration and database operations
- `archive_agent/core/IngestionManager.py` - Parallel file processing and progress tracking
- `archive_agent/core/CliManager.py` - CLI display and logging with multithreading
- `archive_agent/ai/AiManager.py` - AI API interactions and prompts
- `archive_agent/__main__.py` - CLI command definitions
- `archive_agent/standalone/StandaloneOcr.py` - Standalone STRICT OCR processor (lightweight, no Qdrant)

## Testing and Quality Assurance

### Primary Testing Command

**Conda/Miniconda Compatibility Note**: All scripts (`install.sh`, `audit.sh`, `archive-agent.sh`) automatically unset conda environment variables (`CONDA_DEFAULT_ENV` and `CONDA_PREFIX`) to prevent conflicts with `uv`. When conda is active, `uv` incorrectly resolves to the conda Python environment instead of the project's `.venv`, causing missing dependency errors. The `unset` statements at the beginning of each script resolve this automatically without requiring users to deactivate conda.

**ALWAYS run the audit script for ANY code changes or testing:**
```bash
./audit.sh
```

**CRITICAL**: Never run individual pytest commands or partial tests. The audit script is the ONLY approved way to run tests as it performs the complete validation suite:
- Unit tests with pytest
- Type checking with pyright
- Code style checking with pycodestyle (PEP 8)

This ensures all code meets project standards and prevents issues from being missed by partial testing.

### Manual Runtime Verification
For multithreaded components, manual testing is required:
```bash
./archive-agent.sh update --verbose --nocache
```

This tests the concurrent processing and live display system.

**CRITICAL**: The audit script **cannot** validate multithreaded runtime behavior. **Manual verification is required** by running the application and visually confirming that all four core multithreading requirements are met:
1. True parallel processing without serialization
2. Styled logging with custom `RichHandler` formatting
3. Clean live display with stable status and scrolling history
4. Real-time updates without glitches

**Note**: If there are files that have been removed from the watchlist, the script will pause for a confirmation prompt. This is expected behavior. The core concurrency and logging test is successful if the live display runs without errors up to that point.

### Parallel Processing Verification Strategy
When developing or modifying parallel processing components:

**Development Workflow**:
1. Run `./audit.sh` after each change for type checking and formatting
2. Manual runtime testing with `./archive-agent.sh update --verbose --nocache`
3. Visual verification of multithreading display (progress bars, logging, UI stability)
4. Regression testing to ensure identical processing results

**Rollback Strategy**:
- Git commits after each working milestone
- Maintain ability to revert to working state at any point
- Test suite validation before proceeding to next phase

**Critical Testing Requirements**:
- All parallel operations must pass identical behavior tests
- Progress tracking must work without serializing worker threads
- Error isolation must not break entire batch processing

## Development Best Practices

### Code Quality
- Follow existing code conventions and patterns
- Use type hints consistently
- Maintain clean imports and formatting
- Never introduce security vulnerabilities or malicious code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shredEngineer/Archive-Agent](https://github.com/shredEngineer/Archive-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
