---
trigger: always_on
description: These instructions define **HOW** Copilot should process user queries and **WHEN** to read specific guidance files.
---

# General Instructions

These instructions define **HOW** Copilot should process user queries and **WHEN** to read specific guidance files.

## Query Processing

### Core Processing Workflow

**MANDATORY SEQUENCE** for every user request:

1. **Pattern Detection**: Identify which context pattern(s) match the user's request
1. **Guidance Loading**: Read required instruction files (minimum 1000 lines each)
1. **Context Integration**: Apply guidance to user's specific query
1. **Response Generation**: Provide solution following loaded guidance

### Critical Directives

- **BEFORE ANY RESPONSE**: Check context patterns and read matching instruction files
- **PATTERN MATCH INDICATOR**: Start responses with "🔍 Pattern Match: [Pattern Name] - Loading guidance"
- **CONTEXT ANALYSIS**: Always analyze user's prompt, included files, folders, conventions, and patterns
- **NO .copilot-tracking ACCESS**: Never search/index `.copilot-tracking/` unless explicitly requested
- **COMPLETE TERMINAL CAPTURE**: Always call `get_terminal_last_command` immediately after EVERY `run_in_terminal` call
- **MONITOR LONG-RUNNING COMMANDS**: Call `get_terminal_last_command` periodically until command finishes
- **NO BACKGROUND INTERACTION**: Use `IsBackground=false` for commands requiring user interaction or confirmation
- **COMPREHENSIVE TERMINAL ANALYSIS**: Provide complete analysis of terminal output and command results
- **CLEAN CODE COMMENTS**: Remove conflicting code comments; never add thinking-process comments

### Response Quality Standards

- **No assumptions** - Always gather context first
- **Complete solutions** - Don't give up unless truly impossible with available tools
- **Clean output** - Use appropriate edit tools, never print code blocks unless requested
- **Terminal command execution** - Always use `run_in_terminal` with `IsBackground=false` for interactive commands
- **Terminal output capture** - Call `get_terminal_last_command` immediately after every terminal command
- **Long-running command monitoring** - Monitor command progress until completion with periodic output checks
- **Comprehensive terminal analysis** - Analyze and explain terminal output, errors, and command results
- **Comprehensive analysis** - Think creatively and explore workspace thoroughly

## Context Recognition

### Context Pattern Recognition

When user queries match these patterns, **immediately** load the corresponding guidance:

| User Query Context                                             | Required Instruction File                                    | Load Priority |
|----------------------------------------------------------------|--------------------------------------------------------------|---------------|
| Deployment, infrastructure provisioning, IaC deployment        | `.github/instructions/deployment.instructions.md`            | Critical      |
| Getting started, setup, help requests                          | `.github/instructions/getting-started.instructions.md`       | Critical      |
| Architecture decisions, best practices, design guidance        | `.github/instructions/architecture-guidance.instructions.md` | Critical      |
| Terraform files (.tf), IaC configuration, modules              | `.github/instructions/terraform.instructions.md`             | Critical      |
| Configuration parameters, SKUs, pricing, variables, validation | `.github/instructions/configuration.instructions.md`         | Critical      |
| Task implementation, .copilot-tracking files                   | `.github/instructions/task-implementation.instructions.md`   | Critical      |

**Pattern Matching Rules:**

- Multiple patterns can match simultaneously - load ALL relevant guidance
- When uncertain, err on the side of loading additional guidance
- Each loaded file must be read with minimum 1000 lines
- Search for matching context patterns before every change and interaction

### Discovery and Context Gathering Strategy

**Before making any changes**, follow this systematic approach:

#### Query Analysis Process

1. **Semantic Search**: Use semantic search to understand codebase patterns
1. **File Discovery**: Identify relevant files using file_search and grep_search
1. **Context Loading**: Read complete files (prefer large chunks over multiple small reads)
1. **Pattern Validation**: Ensure approach aligns with existing conventions

### Context Integration Rules

- **Always read guidance BEFORE acting** - Never assume or shortcut
- **Read minimum 1000 lines** from each loaded instruction file
- **Follow loaded guidance exactly** - Don't modify or interpret
- **Integrate multiple guidance sources** when patterns overlap
- **Validate changes against loaded patterns** before implementation
- **Think comprehensively** - Consider user prompt, included files, folders, conventions, and workspace patterns

#### File Reading Strategy

| File Type      | When to Read           | Purpose                               |
|----------------|------------------------|---------------------------------------|
| `README.md`    | Any directory context  | Complete component understanding      |
| `variables.tf` | Terraform modules      | Input parameters and validation rules |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/CAIRA](https://github.com/microsoft/CAIRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
