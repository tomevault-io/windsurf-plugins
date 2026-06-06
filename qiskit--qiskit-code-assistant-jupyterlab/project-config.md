---
trigger: always_on
description: This file provides guidance to AI development assistants when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI development assistants when working with code in this repository.

**Supported AI Assistants:**

- IBM Bob
- Claude Code
- GitHub Copilot
- Cursor AI
- Windsurf
- Gemini CLI
- Any AI assistant with codebase context awareness

## Project Overview

qiskit-code-assistant-jupyterlab is a JupyterLab extension that provides AI-powered code completion for quantum computing development using Qiskit. It integrates with LLM APIs (IBM Quantum Cloud or OpenAI-compatible endpoints) to provide intelligent, context-aware code suggestions for Python quantum programs.

### Core Purpose

- Accelerate Qiskit code development with AI-powered completions
- Help developers learn Qiskit best practices through suggestions
- Support migration from old Qiskit to v2.x
- Provide real-time streaming code generation
- Work both with cloud (IBM Quantum) and local (Ollama) deployments

### Key Technologies

- **Extension Platform**: JupyterLab Extension API (v4.3.0+)
- **Language**: TypeScript (strict mode)
- **Backend**: Python server extension
- **LLM Integration**: IBM Quantum Cloud API, OpenAI-compatible APIs
- **Model**: `mistral-small-3.2-24b-qiskit` (via cloud) or other Qiskit Code Assistant LLMs (available through https://huggingface.co/Qiskit/models) via Ollama.
- **Streaming**: Server-Sent Events (SSE) with circuit breaker pattern
- **Testing**: Jest for frontend, pytest for backend

## Architecture

### Component Structure

1. **Extension Entry Point** ([src/index.ts](src/index.ts)):
   - Main plugin initialization
   - Registers completion provider and status bar widget
   - Manages settings and credentials

2. **Completion Provider** ([src/QiskitCompletionProvider.ts](src/QiskitCompletionProvider.ts)):
   - Implements JupyterLab's inline completion interface
   - Handles code context extraction
   - Manages suggestion lifecycle

3. **Status Bar Widget** ([src/StatusBarWidget.ts](src/StatusBarWidget.ts)):
   - Displays model status and credentials
   - Provides user interaction for token/model selection
   - Shows telemetry status

4. **Services** ([src/service/](src/service/)):
   - `api.ts`: Unified API layer and routing
   - `autocomplete.ts`: Code completion service
   - `migration.ts`: Migration-specific service
   - `credentials.ts`: Credential management
   - `token.ts`: API token handling
   - `modelHandler.ts`: Model selection logic

5. **Utilities** ([src/utils/](src/utils/)):
   - `handler.ts`: HTTP request handling and streaming
   - `icons.ts`: UI icon definitions
   - `schema.ts`: Schema type definitions

6. **Python Backend** ([qiskit_code_assistant_jupyterlab/](qiskit_code_assistant_jupyterlab/)):
   - Server extension for backend API
   - Handles server-side processing

### Data Flow

```
User Types Code → JupyterLab Inline Completion Trigger
                           ↓
                  QiskitCompletionProvider
                           ↓
                  Service API (routing)
                           ↓
              autocomplete.ts OR migration.ts
                           ↓
              handler.ts (SSE streaming)
                           ↓
              Inline Suggestion Display
                           ↓
         User Accepts (Tab) or Rejects (Esc)
```

## Key Components

### Configuration Settings

All extension settings are in schema/plugin.json:

- `apiUrl`: API endpoint (default: https://qiskit-code-assistant.quantum.ibm.com)
- `apiToken`: API authentication token
- `selectedCredential`: Selected credential from ~/.qiskit/qiskit-ibm.json
- `selectedModel`: Selected LLM model
- `enableTelemetry`: Opt-in telemetry (default: true)
- `enableStreaming`: Real-time streaming (default: true)
- `streamingDebounceMs`: Debounce delay in milliseconds (default: 500)
- `maxTokens`: Maximum tokens in response (default: 1000)

### Core Files and Directories

- `src/index.ts`: Main extension entry point
- `src/QiskitCompletionProvider.ts`: Completion provider implementation
- `src/StatusBarWidget.ts`: Status bar UI
- `src/service/`: API service layer
- `src/utils/`: Core utilities (HTTP, icons, schema)
- `src/__tests__/`: Frontend test suite
- `qiskit_code_assistant_jupyterlab/`: Python backend
- `package.json`: NPM package manifest
- `pyproject.toml`: Python package configuration
- `schema/`: Settings schema definitions
- `docs/`: User and developer documentation

## Development Guidelines

### Environment Setup

1. **Prerequisites**:
   - JupyterLab 4.3.0+
   - Node.js v18+
   - Python 3.8+
   - Git

2. **Installation**:

   ```bash
   # Clone the repository
   git clone https://github.com/Qiskit/qiskit-code-assistant-jupyterlab.git
   cd qiskit-code-assistant-jupyterlab

   # Install package in development mode
   pip install -e "."

   # Link your development version of the extension with JupyterLab
   jupyter labextension develop . --overwrite

   # Enable server extension
   jupyter server extension enable qiskit_code_assistant_jupyterlab

   # Install JavaScript dependencies and build
   jlpm install
   jlpm build
   ```

3. **Running from Source**:

   ```bash
   # Watch for changes in one terminal
   jlpm watch

   # Run JupyterLab in another terminal
   jupyter lab
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qiskit/qiskit-code-assistant-jupyterlab](https://github.com/Qiskit/qiskit-code-assistant-jupyterlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
