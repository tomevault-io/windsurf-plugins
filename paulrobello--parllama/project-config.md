---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PAR LLAMA is a Terminal User Interface (TUI) application for managing and interacting with Large Language Models (LLMs). Built with Textual and Rich frameworks, it provides a comprehensive interface for:
- Ollama model management (local LLM runtime)
- Multi-provider support (OpenAI, Anthropic, Groq, XAI, OpenRouter, Deepseek, LiteLLM)
- Chat interfaces with image support for vision models
- Session management and prompt customization

## Development Commands

### Setup and Installation
```bash
make setup          # First-time setup with uv
make dev            # Run in development mode with hot reload
make run            # Run the application normally
```

### Code Quality Commands
```bash
make checkall       # Format, lint, and typecheck (run before commits)
make format         # Format code with ruff
make lint           # Lint code with ruff
make typecheck      # Type check with pyright
```

### Testing
```bash
make test           # Run unit tests
```

### Build and Publish
```bash
make package        # Build the package
make test-publish   # Upload to TestPyPI
make publish        # Upload to PyPI
```

## Architecture and Key Components

### Application Structure
- **Main Entry**: `src/parllama/app.py` - Textual application class
- **Screens**: Main, save session, and import screens in `src/parllama/screens/`
- **Views**: Tab-based views (chat, models, prompts, options) in `src/parllama/views/`
- **Widgets**: Reusable UI components for chat, model lists, etc.
- **Message System**: Event-driven communication between components via `src/parllama/messages/`

### Provider System
- Base provider interface in `par_ai_core` dependency
- Providers configured via environment variables and Options screen
- Session config manages provider selection and model parameters
- **Provider Cache Management**: Intelligent per-provider model caching system
  - Configurable cache durations (1 hour to 1 year) per provider in Options screen
  - Smart defaults: Ollama (168h), cloud providers (24-48h), aggregators (24h)
  - Real-time cache status with age, model count, and last refresh time
  - Manual refresh capabilities per provider for immediate cache invalidation
  - Cache logic in `src/parllama/provider_manager.py` with Settings integration

### Data Persistence
- Sessions stored as JSON in `~/.local/share/parllama/`
- Custom prompts and themes in respective subdirectories
- Settings persisted between runs unless `--no-save` flag used
- **File Security**: All file operations use comprehensive validation and secure operations

### Key Technical Details
- **Async Architecture**: Heavy use of asyncio for non-blocking operations
- **Type Safety**: Fully typed with Python type annotations
- **Message Passing**: Components communicate via Textual's message system
- **Theme System**: JSON-based theming with dark/light mode support
- **File Security**: Comprehensive validation system protects against malicious files and ensures data integrity

### File Security System
PAR LLAMA implements a comprehensive file validation and security system to protect against malicious files and ensure data integrity:

#### Core Components
- **FileValidator** (`src/parllama/validators/file_validator.py`): Comprehensive file validation with security checks
- **SecureFileOperations** (`src/parllama/secure_file_ops.py`): Atomic file operations with backup/restore capabilities

#### Security Features
- **Path Security**: Prevents directory traversal attacks and validates file paths
- **File Size Limits**: Configurable limits per file type (JSON: 20MB, Images: 5MB, General: 10MB)
- **Extension Validation**: Whitelist-based file extension checking
- **Content Validation**: Format-specific validation for JSON, images, and ZIP files
- **ZIP Bomb Protection**: Compression ratio checking to prevent zip bombs
- **Atomic Operations**: All file writes use atomic operations with automatic backup/restore
- **Filename Sanitization**: Removes unsafe characters and validates against reserved names

#### Configuration Settings (v0.3.26)
All file validation features are configurable via `settings_manager.py`:
```python
file_validation_enabled: bool = True
max_file_size_mb: float = 10.0
max_image_size_mb: float = 5.0
max_json_size_mb: float = 20.0
max_zip_size_mb: float = 50.0
max_zip_compression_ratio: float = 100.0
validate_file_content: bool = True
sanitize_filenames: bool = True
allowed_json_extensions: list[str] = [".json"]
allowed_image_extensions: list[str] = [".jpg", ".jpeg", ".png", ".gif", ".webp", ".bmp"]
allowed_markdown_extensions: list[str] = [".md", ".markdown", ".txt"]
allowed_zip_extensions: list[str] = [".zip"]
atomic_file_operations: bool = True
create_backup_on_write: bool = True
```

#### Protected Operations
- Session persistence (JSON files)
- Prompt storage and import/export
- Theme loading and management
- Chat input history
- Conversation import/export (Markdown)
- Fabric prompt downloads (ZIP files)
- Image caching and validation
- Settings persistence

## Development Guidelines

### Code Style
- Use type annotations for all functions and methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulrobello/parllama](https://github.com/paulrobello/parllama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
