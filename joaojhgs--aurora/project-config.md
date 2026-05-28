---
trigger: always_on
description: **ALWAYS FOLLOW THESE INSTRUCTIONS FIRST**. Only search for additional context or run extra commands if the information here is incomplete or found to be incorrect.
---

# Aurora Voice Assistant Development Instructions

**ALWAYS FOLLOW THESE INSTRUCTIONS FIRST**. Only search for additional context or run extra commands if the information here is incomplete or found to be incorrect.

Aurora is a Python-based intelligent voice assistant for local automation and productivity. It uses real-time speech-to-text, LLMs, and various productivity tools in a modular, privacy-focused architecture.

## Bootstrap and Environment Setup

**CRITICAL**: Aurora requires Python 3.9-3.11. Python 3.12+ causes dependency conflicts. Check your Python version first:
```bash
python --version  # Must be 3.9.x, 3.10.x, or 3.11.x
# If Python 3.12+: Install Python 3.11 with pyenv or system package manager
```

**Python version validation (run this first):**
```bash
python -c "import sys; print(f'Version: {sys.version}'); print('Compatible' if sys.version_info[:2] in [(3,9), (3,10), (3,11)] else 'INCOMPATIBLE - Use Python 3.9-3.11')"
```

**Set up the development environment:**
```bash
# Install system dependencies (Linux) - takes 1-2 minutes
sudo apt update && sudo apt install -y portaudio19-dev python3-pip python3-venv python3-dev gcc

# Run guided setup (RECOMMENDED) - detects hardware and configures automatically
./setup.sh  # Choose option 3 for Development

# OR manually install development dependencies
pip install -r requirements-dev.txt  # Takes 2-3 minutes, set timeout to 300 seconds
```

**NEVER CANCEL: Setup takes 5-10 minutes on first run. Set timeout to 900+ seconds.**

## Build and Test Commands

**Build the project:**
```bash
# NEVER CANCEL: Full build takes 15-20 minutes. Set timeout to 1800+ seconds.
make setup  # Complete environment setup

# Format code (takes 30-60 seconds)
make format

# NEVER CANCEL: Linting takes 2-3 minutes. Set timeout to 300+ seconds.
make lint

# NOTE: If dependency installation fails due to Python 3.12+:
# Error messages like "piper-phonemize==1.1.0 not found" are expected
# Use Python 3.9-3.11 or run simplified validation commands below
```

**Run tests:**
```bash
# NEVER CANCEL: All tests take 10-15 minutes. Set timeout to 1200+ seconds.
make test

# Run specific test categories (each takes 3-8 minutes)
make unit        # Unit tests - 3-5 minutes
make integration # Integration tests - 5-8 minutes  
make coverage    # Coverage report - 8-12 minutes

# NEVER CANCEL: Full test suite can take 20+ minutes. Set timeout to 1800+ seconds.
pytest  # Run all tests except performance tests
```

**Performance tests (optional):**
```bash
# NEVER CANCEL: Performance tests take 15-30 minutes. Set timeout to 2400+ seconds.
pytest tests/performance
```

## Running Aurora

**Basic application run:**
```bash
# NEVER CANCEL: First run takes 5-10 minutes (downloads models). Set timeout to 900+ seconds.
python main.py

# Run without UI (headless mode)
python main.py  # UI activation controlled by config.json
```

**Configuration:**
- Copy `.env.file` to `.env` and add API keys
- Modify `config.json` for settings (defaults work for most users)
- Model files stored in `chat_models/` and `voice_models/` directories

## Manual Validation Scenarios

**ALWAYS test these scenarios after making changes:**

1. **Python Version Compatibility Check:**
   ```bash
   python -c "import sys; print('✅ Compatible' if sys.version_info[:2] in [(3,9), (3,10), (3,11)] else '❌ INCOMPATIBLE - Use Python 3.9-3.11')"
   # Verify: Shows "✅ Compatible"
   ```

2. **Basic Application Import Test:**
   ```bash
   python -c "from app.config.config_manager import config_manager; print('✅ Basic imports successful')"
   # Verify: No import errors
   # Verify: Configuration loads successfully
   # Expected: May create config.json if missing
   ```

3. **Application Startup Test:**
   ```bash
   python main.py
   # Verify: Application starts without crashes
   # Verify: Configuration loads successfully
   # Verify: No import errors in logs
   # NOTE: Will fail with missing dependencies unless full setup completed
   ```

4. **Development Tools Workflow:**
   ```bash
   make format && make lint
   # Verify: Code formatting applies cleanly
   # Verify: Linting passes or shows expected warnings only
   # NOTE: Some linting errors are expected if dependencies not fully installed
   ```

5. **Test Suite Basic Validation:**
   ```bash
   make unit
   # Verify: Unit tests execute (may skip tests requiring missing dependencies)
   # Verify: No critical import failures
   # Check logs for any failures related to your changes specifically
   ```

## Common Development Workflows

**Adding new features:**
1. Create branch: `git checkout -b feature/your-feature-name`
2. Run setup: `./setup.sh` (choose Development option)
3. Make changes in `app/` directory
4. **ALWAYS run validation:** `make format && make lint && make unit`
5. Test manually: `python main.py`
6. Commit with pre-commit hooks

**Working with dependencies:**
```bash
# Add runtime dependency to pyproject.toml [project.dependencies]
# Add dev dependency to pyproject.toml [project.optional-dependencies.dev]
# Update requirements-*.txt files accordingly
pip install -e .[dev-local-cpu]  # Install in development mode
```

**Before committing:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaojhgs/aurora](https://github.com/joaojhgs/aurora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
