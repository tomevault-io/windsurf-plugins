---
trigger: always_on
description: `GenerateAgents.md` is a Python command-line tool that automates the creation of a comprehensive `AGENTS.md` file for any public GitHub or local code repository. It acts as an automated codebase analyst and technical writer, using the `dspy` framework to programmatically interface with LLMs. The tool clones and analyzes a target codebase to produce a standardized blueprint, enabling AI coding agents to rapidly understand a project's architecture, conventions, and data flow. The primary language 
---

# AGENTS.md — AutogenerateAgentsMD.md

## Project Overview

`GenerateAgents.md` is a Python command-line tool that automates the creation of a comprehensive `AGENTS.md` file for any public GitHub or local code repository. It acts as an automated codebase analyst and technical writer, using the `dspy` framework to programmatically interface with LLMs. The tool clones and analyzes a target codebase to produce a standardized blueprint, enabling AI coding agents to rapidly understand a project's architecture, conventions, and data flow. The primary language is Python (>=3.12).

## Tech Stack

*   **Primary Language:** Python (>=3.12)
*   **Core AI Framework:** `dspy`
*   **LLM Abstraction Layer:** `litellm`
*   **Dependency Management:** `uv`
*   **CLI Framework:** `argparse` (standard library)
*   **Configuration:** `python-dotenv`
*   **Version Control Interaction:** `git` (via `subprocess`)
*   **Testing:** `pytest`

## Architecture

The application follows a modular, stateless pipeline pattern orchestrated by the main CLI entry point.

*   `src/autogenerateagentsmd/cli.py`: The command-line interface entry point. It parses arguments and orchestrates the entire analysis and generation pipeline via the `run_agents_md_pipeline` function.
*   `src/autogenerateagentsmd/modules.py`: Contains the core `dspy.Module` classes (`CodebaseConventionExtractor`, `AgentsMdCreator`, `AntiPatternExtractor`). These modules encapsulate the primary LLM-driven logic for analyzing code and synthesizing the final document.
*   `src/autogenerateagentsmd/signatures.py`: Defines the contracts for LLM interactions using `dspy.Signature`. These signatures specify the expected inputs (e.g., source code) and outputs (e.g., extracted conventions) for each LLM-powered step.
*   `src/autogenerateagentsmd/model_config.py`: Centralizes the configuration for supported LLMs, making it easy to switch between models like Gemini, Claude, and OpenAI.
*   `src/autogenerateagentsmd/utils.py`: Contains helper functions for non-LLM tasks, such as cloning Git repositories, loading files into memory, and other file system operations.
*   `tests/`: The test suite, containing end-to-end and unit tests.
*   `pyproject.toml`: Defines project metadata, dependencies, and the `autogenerateagentsmd` console script entry point.

## Code Style

The project enforces a strict and consistent Python coding style.

*   **Type Hinting**: Type hints are strictly mandatory for all function parameters and return values.

    ```python
    # Good
    def load_source_tree(repo_path: str) -> dict[str, str]:
        # ...

    # Bad
    def load_source_tree(repo_path):
        # ...
    ```

*   **Naming Conventions**:
    *   `snake_case` for functions, methods, and variables (e.g., `run_agents_md_pipeline`).
    *   `PascalCase` for all classes (e.g., `CodebaseConventionExtractor`).
    *   `ALL_CAPS_WITH_UNDERSCORES` for module-level constants.

*   **Import Ordering**: Imports must be grouped at the top of each file in the following order: 1) Standard library, 2) Third-party libraries, 3) Local application imports.

    ```python
    # Good
    import argparse
    import logging
    from pathlib import Path

    import dspy
    from dotenv import load_dotenv

    from .modules import AgentsMdCreator, CodebaseConventionExtractor
    from .utils import clone_repo, load_source_tree
    ```

*   **Formatting**: Use 4-space indentation and maintain a line length between 80-100 characters.

## Anti-Patterns & Restrictions

*   **NEVER use on private codebases**: The tool sends the full source code to third-party LLM APIs. Using it on private, proprietary, or sensitive codebases is a significant security risk. It is designed **exclusively** for public, open-source repositories.
*   **NEVER commit secrets**: Do not commit the `.env` file or any other file containing API keys or secrets to version control.
*   **DO NOT introduce new LLM frameworks**: The project architecture is tightly coupled to `dspy`. Avoid introducing other orchestration frameworks like LangChain or LlamaIndex.
*   **AVOID new end-to-end tests**: E2E tests are slow and costly due to live API calls. Prioritize mocked unit tests for new functionality unless there is a strong justification for an E2E test.

## Database & State Management

The application is entirely **stateless**. It does not use a database or any form of persistent storage between runs. All configuration is loaded at runtime from command-line arguments and the `.env` file. The application's state exists only for the duration of a single execution, primarily as an in-memory dictionary (`source_tree`) holding the target repository's code. The only output is the generated `AGENTS.md` file.

## Error Handling & Logging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [originalankur/GenerateAgents.md](https://github.com/originalankur/GenerateAgents.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
