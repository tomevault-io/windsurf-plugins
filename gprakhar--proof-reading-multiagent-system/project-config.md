---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# IMPORTANT
- Never create a virtual env
- The virtual env folder is "/Users/gadappa/GenAI/genAI/proof_reading_multiagent_system/.virtenv"
- Always first activate the virtual env, before running a python command.
- Always first activate the virtual env, before installing dependencies.

## 1. Project Setup & Environment

### Project Root Directory
**Root Directory**: `/Users/gadappa/GenAI/genAI/proof_reading_multiagent_system`

This is the project root directory where all operations should be performed. Key characteristics:
- Contains the `.virtenv` virtual environment directory
- Contains the main `proof_reading_multiagent_system/` source code directory
- All Python commands, git operations, and development workflows should be run from this directory
- The virtual environment activation path is `source .virtenv/bin/activate` from this directory

### Memory Management (Development Environment)
**Target**: 8GB laptop development environment (production uses Google Cloud Run Functions)

#### Node.js Memory Configuration
```bash
# Add to ~/.zshrc for permanent configuration
export NODE_OPTIONS="--max-old-space-size=3072"  # 3GB heap limit for 8GB laptop
```

#### Memory Issue Troubleshooting
1. **Check Node.js memory limit**: Ensure `NODE_OPTIONS` is set to 3GB
2. **Monitor file sizes**: Keep development Excel files under 5MB when possible
3. **Restart when needed**: Don't hesitate to restart Claude Code if memory usage is high
4. **Browser management**: Close unnecessary tabs to free system memory

## 2. Architecture & Data Flow

### Project Overview
This is a **Proof Reading Multiagent System** built with CrewAI to automate document review and severity assessment for medical/scientific documents. The system uses two specialized agents to review document issues from Excel files and reassess their severity levels using configurable rulesets.

### CrewAI Architecture Pattern
- **Main Entry Point**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/main.py` - Contains execution functions (run, train, replay, test)
- **Crew Definition**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/crew.py` - Uses `@CrewBase` decorator to define agents and tasks
- **Agent Configuration**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/config/agents.yaml` - YAML-based agent definitions
- **Task Configuration**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/config/tasks.yaml` - YAML-based task definitions
- **Custom Tools**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/tools/` - Custom CrewAI tools extending `BaseTool`
- **Logging Infrastructure**: `proof_reading_multiagent_system/src/proof_reading_multiagent_system/utils/` - Comprehensive logging with Google Cloud integration

### Project Directory Structure
```
proof_reading_multiagent_system/
├── logs/                        # Root-level log files directory
│   ├── archive/                 # Archived log files by date
│   ├── proof_reading_system.log # Main system log file
│   └── session_*.log            # Session-specific log files
├── proof_reading_multiagent_system/  # Main project directory
│   ├── data/                    # Data files
│   │   ├── benchmarks/          # Performance benchmark data
│   │   ├── output/              # Generated output files
│   │   └── samples/             # Sample input files for testing
│   ├── docs/                    # Documentation
│   │   ├── api/                 # API documentation
│   │   ├── deployment/          # Deployment guides
│   │   ├── development/         # Development documentation
│   │   └── user-guides/         # User guides and tutorials
│   ├── knowledge/               # CrewAI knowledge base
│   │   └── user_preference.txt  # User context and preferences
│   ├── scripts/                 # Utility scripts
│   │   ├── extract_rules.py     # Rule extraction utilities
│   │   └── performance_benchmark.py # Performance testing
│   ├── src/                     # Source code
│   │   └── proof_reading_multiagent_system/  # Main package
│   │       ├── config/          # Agent and task configurations
│   │       ├── models/          # Data models and schemas
│   │       ├── tools/           # Custom CrewAI tools
│   │       ├── utils/           # Utilities and logging infrastructure
│   │       ├── config.yaml      # Main system configuration
│   │       ├── crew.py          # CrewAI crew definition
│   │       └── main.py          # Main execution entry point
│   ├── tests/                   # Test suite
│   │   ├── fixtures/            # Test fixtures and data
│   │   ├── integration/         # Integration tests
│   │   ├── performance/         # Performance tests
│   │   └── unit/               # Unit tests
│   ├── pyproject.toml          # Project configuration and dependencies
│   └── README.md               # Project-specific README
├── CLAUDE.md                   # Claude Code instructions (this file)
├── requirements.txt            # Python dependencies
└── README.md                   # Root-level README
```

### Data Processing Architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gprakhar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
