---
trigger: always_on
description: > **For AI Assistants**: This document provides comprehensive guidance for understanding and working with the personalUtils codebase.
---

# CLAUDE.md - AI Assistant Guide

> **For AI Assistants**: This document provides comprehensive guidance for understanding and working with the personalUtils codebase.

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Key Concepts](#key-concepts)
- [Development Workflows](#development-workflows)
- [Code Conventions](#code-conventions)
- [Testing](#testing)
- [Configuration](#configuration)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

---

## Overview

**personalUtils** is a Python-based personal utility toolkit featuring:

1. **ChatSystem** - Production-ready GPT-powered chat system with function calling and streaming
2. **Four Specialized AI Agents** - Task execution, transcript analysis, strategic guidance, and framework teaching
3. **12 Utility Tools** - Development utilities automatically integrated as function-calling tools
4. **Agentic Workflows** - Multi-step task planning and execution with reasoning capabilities

### Technology Stack

- **Language**: Python 3.9+
- **AI Models**: OpenAI GPT-4o, GPT-4.1, o3-mini
- **Key Libraries**:
  - `openai>=2.7.1` - OpenAI API integration
  - `pydantic>=2.12.4` - Data validation and settings
  - `rich>=14.2.0` - Terminal UI
  - `tiktoken>=0.12.0` - Token counting
  - `pyyaml>=6.0.2` - Configuration management

### Project Purpose

This repository serves as a personal utility collection combining:
- Reusable development tools (code analysis, API testing, file operations)
- AI-powered chat interface with tool integration
- Specialized AI agents for different cognitive tasks (analysis, strategy, learning, execution)

---

## Repository Structure

```
personalUtils/
├── ChatSystem/              # Core chat system
│   ├── core/               # Core functionality
│   │   ├── chat_engine.py      # OpenAI integration, streaming, function calling
│   │   ├── conversation.py     # Memory management, context handling
│   │   └── config.py           # Pydantic settings and configuration
│   ├── tools/              # Tool integration layer
│   │   ├── tool_adapter.py     # Converts utilities to OpenAI function schemas
│   │   ├── tool_registry.py    # Tool discovery and registration
│   │   └── tool_executor.py    # Safe tool execution with error handling
│   ├── interface/          # User interfaces
│   │   └── cli.py              # Rich-powered interactive CLI
│   ├── examples/           # Example usage scripts
│   └── tests/              # ChatSystem unit tests
│
├── agents/                  # Specialized AI agents
│   ├── agent_manager.py        # Central agent orchestration
│   ├── task_executor/          # General-purpose task execution agent
│   │   ├── executor.py             # Multi-step task execution
│   │   ├── planner.py              # Task planning logic
│   │   └── reasoner.py             # Chain-of-thought reasoning
│   ├── transcript_analyzer/    # Extracts insights from transcripts
│   │   └── analyzer.py
│   ├── trillionaire_futurist/  # Strategic advisor at trillionaire scale
│   │   └── futurist.py
│   └── framework_teacher/      # Teaches through mental models/frameworks
│       └── teacher.py
│
├── tools/                   # 12 utility tools
│   ├── CodeWhisper/            # Python code analysis (LOC, complexity, structure)
│   ├── APITester/              # HTTP API endpoint testing
│   ├── DuplicateFinder/        # Find duplicate files by hash/name
│   ├── SnippetManager/         # Code snippet storage with tags
│   ├── BulkRename/             # Batch file renaming with patterns/regex
│   ├── EnvManager/             # .env file management
│   ├── FileDiff/               # File comparison and diffing
│   ├── GitStats/               # Git repository statistics
│   ├── ImportOptimizer/        # Python import optimization
│   ├── PathSketch/             # Path operations and normalization
│   ├── TodoExtractor/          # Extract TODO/FIXME comments
│   └── DataConvert/            # Convert between JSON, YAML, CSV, XML
│
├── tests/                   # Test suite
│   ├── test_agents.py          # Agent functionality tests
│   └── test_agent_switching.py # Agent switching tests
│
├── config.yaml              # Global configuration (models, tools, agents)
├── .env                     # Environment variables (API keys)
├── .env.example             # Environment template
├── requirements.txt         # Python dependencies
├── requirements-dev.txt     # Development dependencies
├── pytest.ini               # Pytest configuration
├── README.md                # Project overview
├── AGENTS_README.md         # Detailed agent documentation
├── CHATSYSTEM_SETUP.md      # Quick setup guide
└── IMPROVEMENTS.md          # Future improvements and ideas
```

### Key File Locations

| Component | Path | Purpose |
|-----------|------|---------|
| Main entry point | `ChatSystem/__main__.py` | Launch interactive chat |
| Chat engine | `ChatSystem/core/chat_engine.py` | Core OpenAI integration |
| Agent manager | `agents/agent_manager.py` | Agent selection and orchestration |
| Configuration | `config.yaml` | Models, tools, agent settings |
| Environment | `.env` | API keys and secrets |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vkstark) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
