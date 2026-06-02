---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with this agent framework. It serves as both development reference and educational resource.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with this agent framework. It serves as both development reference and educational resource.

## 🏗️ Project Architecture Deep Dive

### Core System Components
```
src/
├── core/
│   ├── generic_agent.py    # Main Agent class - LLM integration and tool orchestration
│   ├── tool_loader.py      # Dynamic tool loading and lifecycle management
│   ├── tool_registry.py    # Tool registration, capability tracking, and discovery
│   ├── tool_sets.py        # Logical tool grouping and system prompt fragments
│   ├── prompt_builder.py   # System prompt construction and command wrapping
│   └── tool_registry.py    # Enhanced registry with pattern matching
├── tools/
│   ├── base_tool.py        # Abstract base classes (Tool, FileSystemTool)
│   ├── filesystem/
│   │   ├── base.py         # FileSystemTool base with security validation
│   │   ├── read_file.py    # File reading with pagination and MIME detection
│   │   ├── write_file.py   # File creation/overwrite with safety checks
│   │   ├── edit_file.py    # Precise text replacement with validation
│   │   ├── grep.py         # Content search with regex support
│   │   └── glob.py         # File discovery with pattern matching
│   └── __init__.py         # Tool package initialization
└── utils/
    └── logger.py           # Structured JSON logging with content filtering
```

### Key Architectural Patterns

#### 1. Tool-Agnostic Design
- **Generic Agent**: `Agent` class works with any tool implementing the `Tool` interface
- **Dynamic Loading**: Tools loaded at runtime based on configuration patterns
- **Capability-Based**: Tools declare capabilities for intelligent tool selection

#### 2. Security First Architecture
- **Workspace Isolation**: All file operations restricted to `filesystem_workspace`
- **Path Validation**: Comprehensive path resolution and traversal prevention
- **Input Sanitization**: All parameters validated before execution

#### 3. Error Resilience System
- **Structured Errors**: `ToolResult` with error codes, details, and recovery suggestions
- **Retry Logic**: Configurable retry attempts with exponential backoff
- **Contextual Recovery**: Error-specific guidance for LLM decision making

#### 4. Smart Content Management
- **Render Levels**: Minimal/Normal/Verbose content filtering for different use cases
- **Pagination Support**: Large file handling with line-based pagination
- **MIME Detection**: Automatic content type recognition

#### 5. Modular Prompt Engineering
- **PromptBuilder**: Dedicated module for system prompt construction and command wrapping
- **Tool Fragments**: Tool-specific guidance integrated dynamically into system prompts
- **Autonomous Execution**: Non-interactive CLI design with complete workflow automation

## 🛠️ Development Commands

### Environment Setup
```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install core dependencies
pip3 install -r requirements.txt

# Install development tools
pip3 install black ruff mypy pytest pytest-cov

# Setup environment
cp .env.example .env
# Edit .env with your API key: API_KEY=your_openai_api_key_here
```

### Testing Framework
```bash
# Run complete test suite
python3 -m pytest tests/ -v

# Test specific components
python3 -m pytest tests/unit/ -v              # Unit tests
python3 -m pytest tests/integration/ -v       # Integration tests
python3 -m pytest tests/e2e/ -v               # End-to-end workflows

# Test with coverage reporting
python3 -m pytest tests/ --cov=src --cov-report=html

# Run specific test file
python3 -m pytest tests/unit/test_generic_agent.py -v
python3 -m pytest tests/unit/test_tool_registry.py -v
python3 -m pytest tests/unit/test_filesystem_tools.py -v
```

### Code Quality & Maintenance
```bash
# Format code with black
black src/

# Lint with ruff
ruff check src/
ruff format src/

# Type checking with mypy
mypy src/

# Run all quality checks
ruff check src/ && black --check src/ && mypy src/

# Security audit
ruff check src/ --select S
```

### Agent Execution & Debugging
```bash
# CLI mode examples
python3 -m src.cli "Your command here" --file target.md
python3 -m src.cli "Analyze project structure" --path .

# Interactive development mode
python3 -m src.cli --interactive

# Debug logging levels
python3 -m src.cli "Debug command" --log-level debug --render-level verbose
python3 -m src.cli "Production run" --log-level warn --render-level minimal

# Custom tool configuration
python3 -m src.cli "Specialized task" --tools filesystem/grep --model gpt-4
```

## 🎓 Educational Focus Areas

### For Agent Beginners
1. **Start with Tool Basics**: Understand `base_tool.py` and the Tool interface
2. **Study Error Handling**: Examine `ToolResult` structure and error codes
3. **Explore Security**: Review path validation in `FileSystemTool`

### For Intermediate Developers
1. **Tool Registration**: Understand `ToolRegistry` and dynamic loading in `tool_loader.py`
2. **LLM Integration**: Study `generic_agent.py` execution loop and prompt construction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kjx-talesofai/md_agent_pro](https://github.com/kjx-talesofai/md_agent_pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
