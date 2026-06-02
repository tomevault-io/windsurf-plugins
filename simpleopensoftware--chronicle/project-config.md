---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chronicle is at the core an AI-powered personal system - various devices, including but not limited to wearables from OMI can be used for at the very least audio capture, speaker specific transcription, memory extraction and retrieval.
On top of that - it is being designed to support other services, that can help a user with these inputs such as reminders, action items, personal diagnosis etc.

This supports a comprehensive web dashboard for management.

**⚠️ Active Development Notice**: This project is under active development. Do not create migration scripts or assume stable APIs. Only offer suggestions and improvements when requested.

**❌ No Backward Compatibility**: Do NOT add backward compatibility code unless explicitly requested. This includes fallback logic, legacy field support, or compatibility layers. Always ask before adding backward compatibility - in most cases the answer is no during active development.

## Initial Setup & Configuration

Chronicle includes an **interactive setup wizard** for easy configuration. The wizard guides you through:
- Service selection (backend + optional services)
- Authentication setup (admin account, JWT secrets)
- Transcription provider configuration (Deepgram or offline ASR)
- LLM provider setup (OpenAI or Ollama)
- Memory provider selection (Chronicle Native with Qdrant or OpenMemory MCP)
- Network configuration and HTTPS setup
- Optional services (speaker recognition, Parakeet ASR)

### Quick Start
```bash
# Run the interactive setup wizard from project root (recommended)
./wizard.sh

# Or use direct command:
uv run --with-requirements setup-requirements.txt python wizard.py

# For step-by-step instructions, see quickstart.md
```

**Note on Convenience Scripts**: Chronicle provides wrapper scripts (`./wizard.sh`, `./start.sh`, `./restart.sh`, `./stop.sh`, `./status.sh`) that simplify the longer `uv run --with-requirements setup-requirements.txt python` commands. Use these for everyday operations.

### Setup Documentation
For detailed setup instructions and troubleshooting, see:
- **[@quickstart.md](quickstart.md)**: Beginner-friendly step-by-step setup guide
- **[@Docs/init-system.md](Docs/init-system.md)**: Complete initialization system architecture and design

### Wizard Architecture
The initialization system uses a **root orchestrator pattern**:
- **`wizard.py`**: Root setup orchestrator for service selection and delegation
- **`backends/advanced/init.py`**: Backend configuration wizard
- **`extras/speaker-recognition/init.py`**: Speaker recognition setup
- **Service setup scripts**: Individual setup for ASR services and OpenMemory MCP

Key features:
- Interactive prompts with validation
- API key masking and secure credential handling
- Environment file generation with placeholders
- HTTPS configuration with SSL certificate generation
- Service status display and health checks
- Automatic backup of existing configurations

## Development Commands

### Backend Development (Advanced Backend - Primary)
```bash
cd backends/advanced

# Start full stack with Docker
docker compose up --build -d

uv run python src/main.py

# Code formatting and linting
uv run black src/
uv run isort src/

# Run tests
uv run pytest
uv run pytest tests/test_memory_service.py  # Single test file

# Run integration tests (local script mirrors CI)
./run-test.sh  # Complete integration test suite

# Environment setup
cp .env.template .env  # Configure environment variables

# Reset data (development)
sudo rm -rf backends/advanced/data/
```

### Running Tests

#### Quick Commands
All test operations are managed through a simple Makefile interface:

```bash
cd tests

# Full test workflow (recommended)
make test              # Start containers + run all tests

# Or step by step
make start             # Start test containers (with health checks)
make test-all          # Run all test suites
make stop              # Stop containers (preserves volumes)

# Run specific test suites
make test-endpoints    # API endpoint tests (~40 tests, fast)
make test-integration  # End-to-end workflows (~15 tests, slower)
make test-infra        # Infrastructure resilience (~5 tests)

# Quick iteration (reuse existing containers)
make test-quick        # Run tests without restarting containers
```

#### Container Management
All container operations automatically preserve logs before cleanup:

```bash
make start             # Start test containers
make stop              # Stop containers (keep volumes)
make restart           # Restart without rebuild
make rebuild           # Rebuild images + restart (for code changes)
make containers-clean  # SAVES LOGS → removes everything
make status            # Show container health
make logs SERVICE=<name>  # View specific service logs
```

**Log Preservation:** All cleanup operations save container logs to `tests/logs/YYYY-MM-DD_HH-MM-SS/`

#### Test Environment

Test services use isolated ports and database:
- **Ports:** Backend (8001), MongoDB (27018), Redis (6380), Qdrant (6337/6338)
- **Database:** `test_db` (separate from production)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimpleOpenSoftware/chronicle](https://github.com/SimpleOpenSoftware/chronicle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
