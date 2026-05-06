---
trigger: always_on
description: Guidance for AI Agents working with this repository.
---

# AGENTS.md

Guidance for AI Agents working with this repository.

## Project Overview

A sophisticated collection of infrastructure templates (boilerplates) with a Python CLI for management. Supports Terraform, Docker, Ansible, Kubernetes, etc. Built with Typer (CLI) and Jinja2 (templating).

## Development Setup

### Running and Testing

```bash
# Run the CLI application
python3 -m cli
# Debugging and Testing commands
python3 -m cli --log-level DEBUG compose list
```

### Production-Ready Testing

For detailed information about testing boilerplates in a production-like environment before release, see **WARP-LOCAL.md** (local file, not in git). This document covers:
- Test server infrastructure and Docker contexts
- Step-by-step testing procedures for Docker Compose, Swarm, and Kubernetes
- Comprehensive testing checklists
- Production release criteria

### Linting and Formatting

Should **always** happen before pushing anything to the repository.

- Use `yamllint` for YAML files
- Use `ruff` for Python code:
  - `ruff check --fix .` - Check and auto-fix linting errors (including unused imports)
  - `ruff format .` - Format code according to style guidelines
  - Both commands must be run before committing

### Project Management and Git

The project is stored in a public GitHub Repository, use issues, and branches for features/bugfixes and open PRs for merging.

**Naming Conventions and Best-Practices:**
- Branches, PRs: `feature/2314-add-feature`, `problem/1249-fix-bug`
- Issues should have clear titles and descriptions, link related issues/PRs, and have appropriate labels like (problem, feature, discussion, question).
- Commit messages should be clear and concise, following the format: `type(scope): subject` (e.g., `fix(compose): correct variable parsing`).

## Architecture

### File Structure

- `cli/` - Python CLI application source code
  - `cli/core/` - Core Components of the CLI application
  - `cli/core/schema/` - JSON schema definitions for all modules
  - `cli/modules/` - Modules implementing technology-specific functions
  - `cli/__main__.py` - CLI entry point, auto-discovers modules and registers commands
- `library/` - Template collections organized by module
  - `library/ansible/` - Ansible playbooks and configurations
  - `library/compose/` - Docker Compose configurations
  - `library/docker/` - Docker templates
  - `library/kubernetes/` - Kubernetes deployments
  - `library/packer/` - Packer templates
  - `library/terraform/` - OpenTofu/Terraform templates and examples
- `archetypes/` - Testing tool for template snippets (archetype development)
  - `archetypes/__init__.py` - Package initialization
  - `archetypes/__main__.py` - CLI tool entry point
  - `archetypes/<module>/` - Module-specific archetype snippets (e.g., `archetypes/compose/`)

### Core Components

- `cli/core/collection.py` - VariableCollection class (manages sections and variables)
  - **Key Attributes**: `_sections` (dict of VariableSection objects), `_variable_map` (flat lookup dict)
  - **Key Methods**: `get_satisfied_values()` (returns enabled variables), `apply_defaults()`, `sort_sections()`
- `cli/core/config.py` - Configuration management (loading, saving, validation)
- `cli/core/display/` - Centralized CLI output rendering package (**Always use DisplayManager - never print directly**)
  - `__init__.py` - Package exports (DisplayManager, DisplaySettings, IconManager)
  - `display_manager.py` - Main DisplayManager facade
  - `display_settings.py` - DisplaySettings configuration class
  - `icon_manager.py` - IconManager for Nerd Font icons
  - `variable_display.py` - VariableDisplayManager for variable rendering
  - `template_display.py` - TemplateDisplayManager for template display
  - `status_display.py` - StatusDisplayManager for status messages
  - `table_display.py` - TableDisplayManager for table rendering
- `cli/core/exceptions.py` - Custom exceptions for error handling (**Always use this for raising errors**)
- `cli/core/library.py` - LibraryManager for template discovery from git-synced libraries and static file paths
- `cli/core/module.py` - Abstract base class for modules (defines standard commands)
- `cli/core/prompt.py` - Interactive CLI prompts using rich library
- `cli/core/registry.py` - Central registry for module classes (auto-discovers modules)
- `cli/core/repo.py` - Repository management for syncing git-based template libraries
- `cli/core/schema/` - Schema management package (**JSON-based schema system**)
  - `loader.py` - SchemaLoader class for loading and validating JSON schemas
  - `<module>/` - Module-specific schema directories (e.g., `compose/`, `terraform/`)
  - `<module>/v*.json` - Version-specific JSON schema files (e.g., `v1.0.json`, `v1.2.json`)
- `cli/core/section.py` - VariableSection class (stores section metadata and variables)
  - **Key Attributes**: `key`, `title`, `toggle`, `needs`, `variables` (dict of Variable objects)
- `cli/core/template.py` - Template Class for parsing, managing and rendering templates
- `cli/core/variable.py` - Variable class (stores variable metadata and values)
  - **Key Attributes**: `name`, `type`, `value` (stores default or current value), `description`, `sensitive`, `needs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evorpewarrior/boilerplates](https://github.com/evorpewarrior/boilerplates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
