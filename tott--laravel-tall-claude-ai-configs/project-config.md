---
trigger: always_on
description: This document provides essential context and quick reference for AI-assisted development in Laravel TALL stack applications.
---

# CLAUDE.md - Laravel TALL Stack AI-Assisted Development Guidelines

This document provides essential context and quick reference for AI-assisted development in Laravel TALL stack applications.

## **🚀 Development Environment**

**Laravel Sail** is used for Docker development. Always prefix commands with `./vendor/bin/sail`:

```bash
# Essential Commands
./vendor/bin/sail up -d          # Start environment  
./vendor/bin/sail down           # Stop environment
./vendor/bin/sail artisan migrate --seed  # Database setup
./vendor/bin/sail npm run dev    # Frontend development
./vendor/bin/sail artisan test   # Run tests
```

**📖 [Complete Commands Reference](docs/reference/laravel-commands.md)**

---

## 🛠️ MCP Server Tools Strategy

**Always use MCP servers for enhanced development:**

### Core Development (Always Available)
- `mcp__serena__*` - Semantic code analysis and intelligent navigation
- `mcp__context7__*` - Up-to-date documentation access
- `mcp__browsermcp__*` - Real-time browser debugging with authenticated sessions

### Quality Assurance (Recommended)
- `mcp__zen__codereview` - Professional code review before PRs
- `mcp__zen__precommit` - Automated quality gates for commits
- `mcp__zen__secaudit` - Security auditing for releases

### Advanced Development (Complex Tasks)
- `mcp__zen__thinkdeep` - Extended reasoning for architectural decisions
- `mcp__zen__debug` - Systematic debugging workflows
- `mcp__zen__consensus` - Multi-model validation for major decisions

**📖 [Complete MCP Server Guides](docs/mcp-servers/)**

---

## 📖 Documentation-First Development

**ALWAYS consult docs/ before starting complex tasks.** The documentation contains:

### Documentation Priority Workflow
```bash
1. **Check CLAUDE.md** → Essential context and quick reference
2. **Consult docs/workflows/** → Understand the process for your task type  
3. **Reference docs/reference/** → Get specific standards, commands, patterns
4. **Engage .claude/agents/** → Delegate complex domain-specific work
5. **Use docs/mcp-servers/** → Optimize tool usage and troubleshooting
```

### When Documentation is Incomplete
```bash
# If docs are missing or outdated, update them FIRST
mcp__serena__write_memory "missing_documentation" "Document what needs to be added"

# Then implement with proper documentation
"Implement [feature] and update docs/workflows/[relevant].md with new patterns discovered"
```

---

## 📊 Smart Tool Selection

**Match tool complexity to task complexity:**
- **Edit/MultiEdit**: Simple, targeted changes and small modifications
- **Serena tools**: Complex refactoring, cross-file analysis, unfamiliar code
- **Zen tools**: Quality assurance, debugging, architectural analysis

**📖 [Complete Development Workflows](docs/workflows/)**

---

## 🏗️ Laravel TALL Stack Architecture Quick Reference

### Core Technology Stack
- **Laravel 12** + **TALL Stack** (Tailwind, Alpine.js, Laravel, Livewire)
- **FilamentPHP** - Admin interfaces (optional)
- **Laravel Sail** - Docker development environment
- **Pest** - PHP testing framework

### Key Application Patterns
- **`app/Livewire/`** - Reactive UI components
- **`app/Services/`** - Business logic services  
- **`app/Models/`** - Eloquent models and relationships
- **`resources/views/livewire/`** - Livewire component templates

**📖 [Complete Architecture Guide](docs/setup/project-architecture.md)**

---

## 🤖 AI Development Guidelines

### Core Development Principles
1. **Documentation First**: Always check `docs/` for detailed information
2. **Pattern Consistency**: Follow existing TALL stack patterns  
3. **Quality Gates**: Use MCP tools for systematic quality assurance
4. **Context Preservation**: Use Serena memory system to document decisions

### Natural Language Development Workflow
```bash
# 1. Context gathering
mcp__serena__get_symbols_overview [relevant_directory]
mcp__serena__search_for_pattern [related_functionality]

# 2. Implementation with quality gates  
mcp__zen__codereview [implemented_feature]
mcp__zen__precommit [validate_changes]

# 3. Knowledge preservation
mcp__serena__write_memory [pattern_name] [architectural_decisions]
```

### Component Architecture Decision Rules
```bash
# Admin functionality + CRUD operations
→ Consider FilamentPHP Resource for rapid development

# User-facing + interactive/real-time  
→ Use Livewire Component

# API endpoints + external integrations
→ Standard Laravel controllers with proper validation
```

### Sub-Agent Coordination Strategy
**Use specialized sub-agents for complex domain-specific work:**

```bash
# Task Complexity Assessment
Simple (1 file, <50 lines)     → Main Agent Only
Moderate (2-3 files)           → Consider specialist agent  
Complex (Multi-system)         → Multi-agent workflow
Architecture/Performance       → Always use specialist

# Example Delegations
"DevOps Specialist: Configure Docker services for [feature]"
"Testing Specialist: Create comprehensive test suite for [component]"
"Security Specialist: Audit authentication system for vulnerabilities"
"Performance Specialist: Optimize database queries in [service]"
```

**📖 [AI Interaction Patterns](docs/reference/ai-interaction-patterns.md)**

---

## 📚 Documentation Contribution Guidelines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tott/laravel-tall-claude-ai-configs](https://github.com/tott/laravel-tall-claude-ai-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
