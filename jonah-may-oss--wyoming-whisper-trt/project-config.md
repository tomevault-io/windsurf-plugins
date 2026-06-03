---
trigger: always_on
description: This file provides guidance for GitHub Copilot coding agent when working on this repository. These instructions help ensure high-quality contributions that align with project standards and practices.
---

# Wyoming Whisper TRT - Copilot Instructions

This file provides guidance for GitHub Copilot coding agent when working on this repository. These instructions help ensure high-quality contributions that align with project standards and practices.

## Project Overview

Wyoming Whisper TRT is a Python-based speech recognition server that optimizes OpenAI Whisper with NVIDIA TensorRT for Home Assistant integration via the Wyoming Protocol. This provides significantly faster inference (~3x faster) while using less memory (~60% less) compared to standard PyTorch Whisper.

**Important**: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Task Suitability and Delegation

### Suitable Tasks for Copilot Agent

This repository is well-suited for Copilot agent assistance on:
- **Bug fixes**: Fixing identified bugs in Python code, Docker configurations, or tests
- **Documentation updates**: Improving README, CONTRIBUTING, or inline documentation
- **Test coverage**: Adding or updating tests for existing functionality
- **Code refactoring**: Improving code structure while maintaining functionality
- **Feature additions**: Implementing well-defined, small to medium features
- **Dependency updates**: Updating package versions in requirements.txt
- **CI/CD improvements**: Enhancing GitHub Actions workflows

### Tasks Requiring Human Review

The following tasks require extra caution and human oversight:
- **Security-related changes**: Authentication, authorization, or secrets management
- **TensorRT model changes**: Core optimization logic that affects performance
- **CUDA/GPU integration**: Low-level GPU code that requires hardware testing
- **Breaking API changes**: Changes that affect Wyoming Protocol compatibility
- **Production configuration**: Changes to Docker images or deployment configs

### Task Expectations

When assigned a task:
1. Read the issue description carefully for context and acceptance criteria
2. Identify the minimal set of files that need modification
3. Make focused, surgical changes - avoid unnecessary refactoring
4. Follow existing code patterns and conventions
5. Test changes thoroughly before marking as complete
6. Update related documentation if API or behavior changes

## Working Effectively

### Bootstrap, Build, and Setup Repository

**CRITICAL**: Always run these commands with LONG timeouts. Builds may take 45+ minutes. NEVER CANCEL long-running operations.

1. **Initialize Git Submodules (Required)**:
   ```bash
   git submodule update --init --recursive
   ```
   - Takes 1-2 minutes. Sets up torch2trt submodule dependency.

2. **Setup Development Environment**:
   ```bash
   chmod +x script/setup
   python script/setup --dev
   ```
   - **NEVER CANCEL**: Takes 45-60 minutes to complete. Set timeout to 90+ minutes.
   - Downloads and installs PyTorch, TensorRT, OpenAI Whisper, Wyoming Protocol, and development tools.
   - Creates `.venv` virtual environment.
   - Installs torch2trt from the git submodule.
   - **Network Requirements**: Requires internet access to PyPI, NVIDIA PyPI, and PyTorch repositories.
   - **Known Issue**: May fail with "Read timed out" errors in restricted network environments.

3. **Alternative Build (Without Dev Dependencies)**:
   ```bash
   python script/setup
   ```
   - Same timing and network requirements as above.

### Validation and Testing

**IMPORTANT**: All validation scripts require dependencies from `script/setup --dev` to be installed first.

1. **Validate Python Syntax (No Dependencies Required)**:
   ```bash
   python -m py_compile wyoming_whisper_trt/__init__.py
   python -m py_compile wyoming_whisper_trt/__main__.py
   python -m py_compile wyoming_whisper_trt/handler.py
   ```
   - Takes 5-10 seconds. Basic syntax validation without external dependencies.

2. **Format Code**:
   ```bash
   python script/format
   ```
   - Takes 10-30 seconds. Runs black and isort formatters.
   - **Requires**: Development dependencies installed via `script/setup --dev`

3. **Lint Code**:
   ```bash
   python script/lint
   ```
   - Takes 1-3 minutes. Runs black, isort, flake8, pylint, and mypy.
   - **NEVER CANCEL**: Set timeout to 10+ minutes for large codebases.
   - **Requires**: Development dependencies installed via `script/setup --dev`

4. **Validate Docker Configuration**:
   ```bash
   docker compose config
   ```
   - Takes 5-10 seconds. Validates docker-compose.yaml syntax and structure.

5. **Run Tests**:
   ```bash
   python script/test
   ```
   - **NEVER CANCEL**: Takes 10-15 minutes. Set timeout to 30+ minutes.
   - Tests Wyoming Protocol integration and speech recognition functionality.
   - Downloads tiny-int8 model if not present (requires ~200MB download).
   - **Requires**: Full dependencies installed via `script/setup --dev`

6. **Package for Distribution**:
   ```bash
   python script/package
   ```
   - Takes 1-2 minutes. Creates wheel distribution in `dist/` directory.

### Running the Application

1. **Local Development Run**:
   ```bash
   python script/run --help
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jonah-May-OSS/wyoming-whisper-trt](https://github.com/Jonah-May-OSS/wyoming-whisper-trt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
