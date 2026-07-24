---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Claude Code Advanced Patterns** repository - a comprehensive package providing production-ready agents, hooks, commands, and workflow patterns for sophisticated AI-assisted development. The project contains no executable code but rather configuration files, documentation, and templates that extend Claude Code's capabilities.

## Essential Commands

### Repository Management
```bash
# Clone repository
git clone https://github.com/aws-samples/anthropic-on-aws.git
cd anthropic-on-aws/advanced-claude-code-patterns

# Install Claude Code (if not already installed)
npm install -g @anthropic-ai/claude-code
# OR 
curl -fsSL claude.ai/install.sh | bash

# Set up project structure
mkdir -p .claude/{agents,commands,hooks}
```

### Component Installation
```bash
# Install agents globally (available in all projects)
cp agents/*.md ~/.claude/agents/

# Install agents for current project only
cp agents/*.md .claude/agents/

# Install slash commands globally
cp commands/**/*.md ~/.claude/commands/

# Install hooks globally
cp hooks/*.{json,py} ~/.claude/hooks/

# Install MCP servers for integrations
claude mcp add github -- npx @modelcontextprotocol/server-github
claude mcp add filesystem -- npx @modelcontextprotocol/server-filesystem .
```

### Testing Components
```bash
# Test agent availability
claude "List available agents"

# Test slash command (use tab completion)
claude /[TAB]

# Verify MCP server connectivity
claude mcp list
claude --mcp-debug
```

## Architecture & Component Structure

### Core System Architecture
The repository implements a **distributed AI orchestration system** with four main component types:

1. **Agents** (`agents/`) - Specialized AI assistants for specific tasks (23 production agents)
2. **Commands** (`commands/`) - Slash commands for workflow orchestration (19 commands with argument support)
3. **Hooks** (`hooks/`) - Lifecycle event automation for quality gates and CI/CD (12 configurations)
4. **Templates** (`templates/`) - Project-specific CLAUDE.md configurations (5 templates)

### Agent Orchestration Pattern
```bash
# Single agent deployment
@security-reviewer

# Parallel agent orchestration (key architectural pattern)
@docs-tutorial-agent @docs-howto-agent @docs-reference-agent @docs-explanation-agent

# Cross-functional team deployment
@architect @security-reviewer @qa-engineer @deployment-agent
```

### Command Structure with Arguments
```bash
# Documentation creation with smart routing
/docs-create "authentication system" --complete    # Creates all 4 doc types
/docs-create "API endpoints" --working            # Creates how-to + reference
/docs-create "concepts" --understanding           # Creates explanation + reference

# EPCC workflow commands
/epcc/epcc-explore "user authentication"          # Analysis phase
/epcc/epcc-plan                                    # Planning phase
/epcc/epcc-code                                    # Implementation phase
/epcc/epcc-commit                                  # Commit with message

# TDD workflow commands
/tdd/tdd-feature "user login"                     # TDD feature development
/tdd/tdd-bugfix "authentication timeout"          # TDD bug fixing
```

## Key Design Patterns

### 1. Agent Specialization & Composition
Each agent has a single responsibility with specific tools and expertise:

- **Documentation Agents**: `docs-tutorial-agent`, `docs-howto-agent`, `docs-reference-agent`, `docs-explanation-agent`
- **Architecture Agents**: `architect`, `system-designer`, `architecture-documenter` 
- **Security & Quality**: `security-reviewer`, `qa-engineer`, `test-generator`
- **Performance**: `performance-profiler`, `optimization-engineer`
- **Agile Roles**: `scrum-master`, `product-owner`, `business-analyst`

### 2. Smart Model Selection Strategy
- **Sonnet (Default)**: 80% of operations - testing, documentation, simple refactoring
- **Opus (Complex)**: 20% of operations - security analysis, performance optimization, architecture

This achieves **70% cost reduction** while maintaining quality.

### 3. Diataxis Documentation Framework
The repository implements the complete Diataxis framework for documentation:

- **Tutorial**: Learning-oriented, hands-on guidance
- **How-to**: Problem-solving, task-oriented guides  
- **Reference**: Information lookup, technical specifications
- **Explanation**: Understanding-oriented, conceptual depth

### 4. EPCC (Explore-Plan-Code-Commit) Workflow
A systematic development methodology:
1. **Explore**: Analyze codebase and requirements
2. **Plan**: Create implementation strategy
3. **Code**: Execute development with agents
4. **Commit**: Generate semantic commit messages

## Working with Components

### Agent Development
When creating or modifying agents:
- Keep agents under 150 lines
- Start with Quick Reference section immediately after frontmatter
- Include specific activation instructions (5-6 lines max)
- Define clear Core Identity with role and principles
- Use Behavioral Contract (ALWAYS/NEVER) for consistency
- Select minimum necessary tools
- Include working code examples

### Hook Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/anthropic-on-aws](https://github.com/aws-samples/anthropic-on-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
