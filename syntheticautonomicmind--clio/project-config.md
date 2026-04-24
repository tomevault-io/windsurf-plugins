---
trigger: always_on
description: **Purpose:** Technical reference for CLIO development (methodology in .clio/instructions.md)
---

# AGENTS.md

**Version:** 3.1  
**Date:** 2026-04-18  
**Purpose:** Technical reference for CLIO development (methodology in .clio/instructions.md)

---

## Project Overview

**CLIO** (Command Line Intelligence Orchestrator) is an AI-powered development assistant built in Perl.

- **Language:** Perl 5.32+
- **Architecture:** Tool-calling AI assistant with terminal UI
- **Philosophy:** The Unbroken Method (see .clio/instructions.md)

---

## Quick Setup

```bash
# Install dependencies
# Run CLIO (no dependencies to install - pure core Perl)
./clio --new

# Debug mode
./clio --debug --new

# Quick test
./clio --input "test query" --exit
```

---

## Architecture

```
User Input
    |
    v
Terminal UI (Chat.pm)
    |
    v
AI Agent (APIManager -> Provider)
    |
    v
Tool Selection (WorkflowOrchestrator)
    |
    v
Tool Execution (ToolExecutor)
    |
    +-- file_operations (17 operations)
    +-- version_control (git + worktrees)
    +-- terminal_operations (shell exec)
    +-- memory_operations (store/recall/LTM)
    +-- todo_operations (task management)
    +-- web_operations (search/fetch)
    +-- code_intelligence (search/analyze)
    +-- interact (checkpoints)
    +-- apply_patch (diff-based editing)
    +-- remote_execution (SSH + parallel) [requires: enable_remote config]
    +-- agent_operations (multi-agent) [requires: enable_subagents config]
    +-- MCPBridge (infrastructure: dynamically exposes MCP server tools)
    |
    v
Result Processing
    |
    v
Markdown Rendering (Markdown.pm)
    |
    v
Terminal Output (with color/theme)
```

---

## Directory Structure

| Path | Purpose |
|------|---------|
| `lib/CLIO/Core/` | System core (APIs, workflow, config) |
| `lib/CLIO/Core/API/` | APIManager sub-modules (ResponseHandler, MessageValidator, etc.) |
| `lib/CLIO/Tools/` | AI-callable tools |
| `lib/CLIO/UI/` | Terminal UI (Chat, Markdown, Theme, Commands) |
| `lib/CLIO/Session/` | Session management |
| `lib/CLIO/Memory/` | Context/memory system (YaRN, TokenEstimator) |
| `lib/CLIO/Profile/` | User personality profile (Analyzer, Manager) |
| `lib/CLIO/Protocols/` | Complex workflows (Architect, Editor, Puppeteer) |
| `lib/CLIO/Providers/` | Direct API providers (Google) |
| `lib/CLIO/Coordination/` | Multi-agent coordination (Broker, Client) |
| `lib/CLIO/MCP/` | Model Context Protocol (servers, transports, OAuth) |
| `lib/CLIO/Security/` | Auth/authz |
| `lib/CLIO/Logging/` | Structured logging |
| `lib/CLIO/Compat/` | Compatibility layers (Terminal) |
| `lib/CLIO/Util/` | Utilities (PathResolver, TextSanitizer, JSON, YAML) |
| `lib/CLIO/Spec/` | OpenSpec integration |
| `docs/` | User/dev documentation |
| `tests/unit/` | Single module tests |
| `tests/integration/` | Cross-module tests |

**Key Files:**

- `clio` - Main executable
- `lib/CLIO/Core/WorkflowOrchestrator.pm` - Tool orchestration
- `lib/CLIO/Core/APIManager.pm` - AI provider integration
- `lib/CLIO/UI/Chat.pm` - Terminal interface
- `lib/CLIO/Core/ToolExecutor.pm` - Tool invocation
- `lib/CLIO/Tools/FileOperations.pm` - File system operations

**Investigate, don't assume:** Use `git log --oneline -20`, `find lib -name "*.pm"`, read actual code.

---

## Model Selection

**Use MiniMax for all sub-agents:**
```
agent_operations(operation: "spawn", task: "...", working_dir: "./CLIO", model: "minimax/minimax-m2.7")
```

MiniMax-M2.7 via MiniMax is the recommended default for all standard tasks: investigation, QA, implementation, code review, refactoring, documentation.

---

## Code Style

**Perl Conventions:**

- Perl 5.32+ with `use strict; use warnings; use utf8;`
- **UTF-8 encoding** for all files
- **4 spaces** indentation (never tabs)
- **POD documentation** for all modules
- **Minimal CPAN deps** (prefer core Perl)

**Module Template:**

```perl
package CLIO::Module::Name;

use strict;
use warnings;
use utf8;
binmode(STDOUT, ':encoding(UTF-8)');
binmode(STDERR, ':encoding(UTF-8)');

=head1 NAME

CLIO::Module::Name - Brief description

=head1 DESCRIPTION

Detailed description of module purpose and behavior

=head1 SYNOPSIS

    use CLIO::Module::Name;
    
    my $obj = CLIO::Module::Name->new();
    $obj->method();

=cut

# Implementation...

1;  # MANDATORY: End every .pm file with 1;
```

**Debug Logging:**

```perl
use CLIO::Core::Logger qw(log_debug log_info log_warning log_error);

# Logger functions handle level-checking internally:
log_debug('ModuleName', 'detailed message');
log_info('ModuleName', 'informational message');
log_warning('ModuleName', 'something unexpected');
log_error('ModuleName', 'something failed: %s', $error);
```

---

## Module Naming Conventions

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `CLIO::Core::` | System core | APIManager, WorkflowOrchestrator, ToolExecutor |
| `CLIO::Core::API::` | APIManager sub-modules | ResponseHandler, MessageValidator, StreamProcessor |
| `CLIO::Tools::` | AI-callable tools | FileOperations, VersionControl, TerminalOperations |
| `CLIO::UI::` | Terminal interface | Chat, Markdown, Theme, ToolOutputFormatter |
| `CLIO::UI::Commands::` | Slash command handlers | API, Session, Config, Project |
| `CLIO::Session::` | Session management | Manager, State, TodoStore, ToolResultStore |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SyntheticAutonomicMind/CLIO](https://github.com/SyntheticAutonomicMind/CLIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
