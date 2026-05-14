---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DevSync** is a CLI tool for AI-powered config distribution across AI coding assistants. It uses LLM intelligence to extract practices from projects and adapt them to recipients' existing setups — supporting 23+ AI tools including Claude Code, Cursor, Windsurf, GitHub Copilot, Kiro, Roo Code, Cline, and Codex.

**CLI entry point:** `devsync` (installed via `pip install devsync`)

## Architecture

### Core Concepts

1. **AI-Powered Extraction**: LLM reads a project's rules, MCP configs, and commands to produce abstract practice declarations (not file copies)
2. **AI-Powered Installation**: LLM adapts incoming practices to recipient's existing setup with intelligent merging and conflict resolution
3. **Project-Level Installation**: All installations are project-specific, stored in tool-specific directories (`.cursor/rules/`, `.claude/rules/`, `.kiro/steering/`, etc.)
4. **Installation Tracking**: Tracked in `<project-root>/.devsync/packages.json` for packages
5. **Graceful Degradation**: No API key? Extract copies files verbatim, install uses file-copy mode. `--no-ai` flag forces this explicitly
6. **v1 Backward Compatibility**: Old `ai-config-kit-package.yaml` packages install via file-copy mode. v2 `devsync-package.yaml` adds `practices` section for AI-native content

### Package Structure

```
devsync/
├── ai_tools/          # AI tool integrations and detection (23+ tools)
│   ├── base.py       # Abstract AITool base class
│   ├── claude.py     # Claude Code (.claude/rules/*.md)
│   ├── cursor.py     # Cursor (.cursor/rules/*.mdc)
│   ├── cline.py      # Cline (.clinerules/*.md)
│   ├── kiro.py       # Kiro (.kiro/steering/*.md)
│   ├── roo.py        # Roo Code (.roo/rules/*.md)
│   ├── winsurf.py    # Windsurf (.windsurf/rules/*.md)
│   ├── codex.py      # OpenAI Codex CLI (AGENTS.md sections)
│   ├── copilot.py    # GitHub Copilot (.github/instructions/*.md)
│   ├── anteroom.py   # Anteroom (ANTEROOM.md sections)
│   └── detector.py   # Tool detection logic
├── cli/               # Typer CLI commands (v2 — 6 commands)
│   ├── main.py       # CLI app definition (setup, tools, extract, install, list, uninstall)
│   ├── setup.py      # Configure LLM provider
│   ├── extract.py    # AI extraction command
│   ├── install_v2.py # AI-powered install command
│   ├── list_v2.py    # Simplified list command
│   ├── uninstall.py  # Uninstall from projects
│   └── tools.py      # List detected AI tools
├── core/              # Core business logic
│   ├── models.py     # Data models (Instruction, Repository, etc.)
│   ├── practice.py   # PracticeDeclaration, MCPDeclaration, CredentialSpec
│   ├── extractor.py  # AI extraction engine
│   ├── adapter.py    # AI adaptation/merge engine
│   ├── package_manifest_v2.py # v2 manifest parser (v1 compat)
│   ├── mcp_credential_prompter.py # MCP credential prompting
│   ├── repository.py # Parse ai-config-kit.yaml
│   ├── git_operations.py # Git clone/pull operations
│   ├── pip_utils.py  # Pip package validation, detection, installation
│   ├── checksum.py   # File integrity checking
│   ├── conflict_resolution.py # Handle file conflicts
│   ├── component_detector.py # Multi-tool component detection and filtering
│   └── capability_registry.py # AI tool capability metadata
├── llm/               # LLM provider abstraction (HTTP-only, no SDK deps)
│   ├── provider.py   # Abstract LLMProvider, LLMResponse, resolve_provider()
│   ├── anthropic.py  # Anthropic Claude (HTTP via httpx)
│   ├── openai_provider.py # OpenAI (HTTP via httpx)
│   ├── openrouter.py # OpenRouter (HTTP via httpx)
│   ├── config.py     # API key resolution, ~/.devsync/config.yaml
│   ├── prompts.py    # All prompt templates
│   └── response_models.py # Structured response types
├── storage/           # Data persistence
│   ├── tracker.py    # InstallationTracker for installations.json
│   └── package_tracker.py # PackageTracker for packages.json
└── utils/             # Utilities
    ├── project.py    # Project root detection
    └── logging.py    # Logging configuration
```

### Key Data Models

From `devsync/core/models.py`:

#### Instructions
- **Instruction**: Single instruction file with name, description, content, file_path, tags, checksum
- **InstructionBundle**: Group of related instructions
- **Repository**: Instruction repository with instructions and bundles
- **InstallationRecord**: Tracks installed instruction with ai_tool, source_repo, installed_path, scope
- **LibraryInstruction**: Instruction in library (downloaded but not installed)
- **LibraryRepository**: Downloaded repository in library

#### Packages
- **Package**: Configuration package containing multiple components (instructions, MCP servers, hooks, commands, resources)
- **PackageComponents**: Container for all components in a package with methods to count and manage them
- **ComponentType**: Enum for component types (INSTRUCTION, MCP_SERVER, HOOK, COMMAND, RESOURCE)
- **InstalledComponent**: Tracks installed component with type, name, path, checksum, and status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [troylar/devsync](https://github.com/troylar/devsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
