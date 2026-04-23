---
trigger: always_on
description: Agent SOP provides natural language workflows that enable AI agents to perform complex, multi-step tasks with consistency and reliability. This is a Python package that implements the Model Context Protocol (MCP) for AI assistant integration and supports conversion to Anthropic Skills format.
---

# AGENTS.md - AI Assistant Context for Agent SOP Project

## Project Overview
Agent SOP provides natural language workflows that enable AI agents to perform complex, multi-step tasks with consistency and reliability. This is a Python package that implements the Model Context Protocol (MCP) for AI assistant integration and supports conversion to Anthropic Skills format.

## Directory Structure

```
agent-sop/
├── python/strands_agents_sops/    # Main Python package
│   ├── __main__.py               # CLI entry point (mcp|skills|rule commands)
│   ├── mcp.py                    # MCP server implementation
│   ├── skills.py                 # Anthropic Skills conversion
│   └── utils.py                  # Shared SOP utilities
├── agent-sops/                   # Source SOP definitions
│   ├── codebase-summary.sop.md   # Documentation generation workflow
│   ├── code-assist.sop.md        # TDD implementation workflow
│   ├── code-task-generator.sop.md # Task breakdown workflow
│   └── pdd.sop.md               # Prompt-driven development workflow
├── python/tests/                 # Comprehensive test suite
├── spec/                         # SOP format specification
└── rules/                        # SOP authoring guidelines
```

## Development Patterns

### SOP Format Structure
All SOPs follow this standardized format:
```markdown
# SOP Title

## Overview
Brief description of what this SOP accomplishes and when to use it.

## Parameters
- **required_param** (required): Description of required input
- **optional_param** (optional, default: value): Description with default

## Steps
### 1. Step Name
Description of what this step accomplishes.

**Constraints:**
- You MUST perform required actions
- You SHOULD follow recommended practices
- You MAY include optional enhancements
- You MUST NOT perform prohibited actions

## Examples
Concrete usage examples showing input and expected outcomes.

## Troubleshooting
Common issues and their solutions.
```

### RFC 2119 Constraint Keywords
- **MUST**: Required actions that cannot be skipped
- **MUST NOT**: Prohibited behaviors, forbidden actions
- **SHOULD**: Recommended practices for optimal results
- **SHOULD NOT**: Not recommended, but not forbidden
- **MAY**: Optional enhancements or alternatives

### Parameter Definition Pattern
```markdown
- **parameter_name** (required|optional, default: value): Description
```

## Core SOPs

1. **codebase-summary**: Analyzes codebases and generates comprehensive documentation
2. **code-assist**: TDD-based implementation with Explore, Plan, Code, Commit workflow
3. **code-task-generator**: Breaks down requirements into manageable tasks
4. **pdd**: Prompt-driven development methodology

## Testing and Quality Assurance

### Running Tests
```bash
# Navigate to python package directory
cd python

# Use hatch to run manages testing environment
hatch test
```

### Code Quality Standards
```bash
# Format code
hatch run format

# Check code quality
hatch run lint

# Auto-fix linting issues
hatch run lint-fix

# Clean build artifacts
hatch run clean
```

## Integration Patterns for AI Assistants

### MCP Integration
```bash
# Start MCP server with built-in SOPs
strands-agents-sops mcp

# Include custom SOPs (sops in path must have `.sop.md` postfix)
strands-agents-sops mcp --sop-paths ~/my-sops:/team/sops
```

**MCP Client Configuration Example:**
```json
{
  "mcpServers": {
    "agent-sops": {
      "command": "strands-agents-sops", 
      "args": ["mcp", "--sop-paths", "~/custom-sops"]
    }
  }
}
```

### Skills Integration
```bash
# Generate Skills format for Claude
strands-agents-sops skills --output-dir ./skills

# Include custom SOPs in Skills generation
strands-agents-sops skills --sop-paths ~/my-sops --output-dir ./skills
```

### Cursor IDE Integration
```bash
# Generate Cursor commands from built-in SOPs (default: .cursor/commands)
strands-agents-sops commands --type cursor

# Specify custom output directory
strands-agents-sops commands --type cursor --output-dir .cursor/commands

# Include custom SOPs in commands generation
strands-agents-sops commands --type cursor --sop-paths ~/my-sops --output-dir .cursor/commands
```

**Usage in Cursor:**
1. Generate commands: Run `strands-agents-sops commands --type cursor` in your project root
2. Execute workflows: In Cursor chat, type `/` followed by the command name (e.g., `/code-assist.sop`)
3. Provide parameters: When prompted, provide the required parameters for the workflow

**Note:** Cursor commands don't support explicit parameters, so the AI will prompt you for required inputs when you execute a command. The generated commands include parameter documentation to guide this interaction.

### Python Integration
```python
from strands import Agent
from strands_tools import editor, shell
from strands_agents_sops import code_assist

# Create agent with SOP as system prompt
agent = Agent(
    system_prompt=code_assist,
    tools=[editor, shell]
)

# Execute SOP
agent("Start code-assist sop")
```

## Build and Distribution Process

### Package Building

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-agents/agent-sop](https://github.com/strands-agents/agent-sop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
