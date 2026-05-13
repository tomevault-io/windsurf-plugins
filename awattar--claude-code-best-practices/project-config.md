---
trigger: always_on
description: This is a documentation repository focusing on Claude Code best practices, patterns, and real-world usage examples. The project serves as a comprehensive guide for developers looking to integrate Claude Code into their terminal-based coding workflows effectively.
---

# Claude Code Best Practices - Project Guide

## Overview

This is a documentation repository focusing on Claude Code best practices, patterns, and real-world usage examples. The project serves as a comprehensive guide for developers looking to integrate Claude Code into their terminal-based coding workflows effectively.

### Project Type

Documentation / Knowledge Repository with Custom Claude Code Commands.

### Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd claude-code-best-practices

# Read the main documentation
cat README.md

# View available custom commands
ls .claude/commands/

# Get help with custom commands
/help-commands     # View all available commands and usage

# Use custom commands (examples)
/commit            # Create conventional commits
/custom-init       # Initialize CLAUDE.md for any project
/issue <number>    # Work on GitHub issues
/reviewpr <number> # Review pull requests
/test <scope>      # Run and improve tests
```

## Architecture

### Project Structure

```
claude-code-best-practices/
├── LICENSE                      # MIT License
├── README.md                    # Main documentation with best practices
├── .gitmessage                  # Git commit message template
├── .github/                     # GitHub templates and workflows
│   ├── pull_request_template.md # Standardized PR template
│   └── COMMIT_CONVENTION.md     # Commit best practices guide
└── .claude/                     # Claude Code configuration
    ├── commands/                # Custom slash commands
    │   ├── commit.md            # Conventional commit helper
    │   ├── custom-init.md       # CLAUDE.md generation command
    │   ├── help-commands.md     # Command help and usage guide
    │   ├── issue.md             # GitHub issue workflow
    │   ├── reviewpr.md          # Pull request review tool
    │   └── test.md              # Test suite management
    └── agents/                  # Specialized AI agents
        ├── general-backend-developer.md
        ├── general-code-quality-debugger.md
        ├── general-devops.md
        ├── general-frontend-developer.md
        ├── general-fullstack-developer.md
        ├── general-qa.md
        ├── general-solution-architect.md
        ├── general-technical-project-lead.md
        └── general-technical-writer.md
```

### Content Organization

The project follows a documentation-first approach with integrated tooling:

- **`README.md`**: Primary content with comprehensive Claude Code guidance.
- **`LICENSE`**: MIT license for open-source usage.
- **`.gitmessage`**: Git commit message template with conventional format.
- **`.github/`**: GitHub templates and workflow configurations.
- **`.claude/commands/`**: Custom workflow commands for Claude Code users.
- **`.claude/agents/`**: Specialized AI agents that enhance command capabilities.

## Technology Stack

### Core Technologies

- **Documentation**: Markdown.
- **Version Control**: Git.
- **Claude Code**: Custom commands, workflows, and specialized AI agents.

### Dependencies

- **GitHub CLI (`gh`)**: Required for issue and PR management commands.
- **Browser Automation**: Puppeteer/Playwright/Selenium (project-dependent).
- **Testing Frameworks**: Varies by project (Jest, pytest, RSpec, etc.).

## Key Features

### 1. Comprehensive Best Practices Guide

- **Location**: `README.md:1-76`
- **Purpose**: Practical guidance for Claude Code usage.
- **Sections**: 
  - Setup tips and environment configuration.
  - Prompt design and context handling.
  - Testing and debugging workflows.
  - Git integration patterns.
  - Hooks and automation.
  - Safety and control measures.

### 2. Custom Claude Code Commands

- **Location**: `.claude/commands/`
- **Purpose**: Streamline common development workflows.
- **Help**: Use `/help-commands` - @.claude/commands/help-commands.md for detailed usage information.

#### Available Commands

- `/custom-init` - `CLAUDE.md` Generator.
- `/commit` - Conventional Commits.
- `/help-commands` - Command Help and Usage Guide.
- `/issue` - GitHub Issue Workflow.
- `/reviewpr` - Pull Request Review.
- `/test` - Test Suite Management.

For detailed information about each command, including usage examples, features, and best practices, use `/help-commands` - @.claude/commands/.

#### Template Integration

Commands now reference standardized templates:

- **Commit messages**: @.gitmessage - Four format variants (single-line, multiline, parent/child, post-review).
- **Pull requests**: @.github/pull_request_template.md - Structured PR format.
- **Commit conventions**: @.github/COMMIT_CONVENTION.md - Best practices guide.

#### Agent Integration

Commands leverage specialized AI agents to provide expert-level capabilities across different domains:

**Core Agents:**
- **general-purpose** - Complex multi-step analysis, file searching, and task coordination
- **general-solution-architect** - Architecture analysis, technology stack decisions, and design patterns
- **general-technical-writer** - Documentation creation, formatting, and content organization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awattar/claude-code-best-practices](https://github.com/awattar/claude-code-best-practices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
