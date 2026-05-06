---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **template repository** for Claude Code local todo system. It provides:

- **Slash Commands**: Context-aware command definitions for development workflows
- **Local Todo System**: File-based project management with hierarchical task organization
- **Project Templates**: Context-aware templates for different project types
- **Zero Installation**: Just copy `.claude/` directory to any project and start using

This repository contains a complete Claude Code setup that works by simply copying the `.claude/` directory to any project.

## Repository Structure

```
.claude/
├── commands/
│   ├── plan/                    # Planning and design slash commands
│   │   ├── brainstorm.md        # /project:plan:brainstorm - Extended thinking sessions
│   │   ├── prd.md              # /project:plan:prd - Product Requirements Documents
│   │   ├── feature.md          # /project:plan:feature - Technical feature planning
│   │   └── tasks.md            # /project:plan:tasks - Task breakdown with hierarchy
│   ├── do/                     # Execution slash commands
│   │   ├── task.md             # /project:do:task - Execute specific tasks
│   │   ├── commit.md           # /project:do:commit - Git commit workflow
│   │   ├── changelog.md        # /project:do:changelog - Generate changelogs
│   │   └── create-worktrees.md # /project:do:create-worktrees - Git worktree management
│   ├── setup/                  # Setup and configuration commands
│   │   └── smart-merge.md      # /project:setup:smart-merge - Intelligent file merging
│   └── project.md              # /project:current - Show current context
├── templates/                  # Context-aware project templates
│   ├── base/                   # Universal templates
│   ├── web-app/               # React/Next.js optimized templates
│   ├── api-service/           # FastAPI/Express optimized templates
│   ├── cli-tool/              # CLI application templates
│   └── saas-platform/         # Multi-service platform templates
├── todo/                      # Local todo system (auto-generated)
│   ├── templates/             # Template files for PRDs, features, tasks
│   ├── prd-001-example/       # Product Requirements Documents
│   │   ├── prd.md
│   │   ├── status.md
│   │   └── tasks/             # Hierarchical task breakdown
│   └── feature-002-example/   # Technical features
│       ├── feature.md
│       ├── status.md
│       └── tasks/
├── contexts/                  # Language-specific coding standards
│   ├── python.md             # Python development guidelines (uv, FastAPI)
│   ├── typescript.md         # TypeScript development guidelines (bun, TanStack)
│   └── react.md              # React + TypeScript guidelines (2025 best practices)
├── archive/                   # Archived systems
│   ├── github-commands/       # Former GitHub Issues workflow commands
│   └── github-utils/          # Former GitHub Projects automation scripts
├── utils/                     # Current utilities
│   ├── merge-mcp.sh          # MCP configuration merging
│   └── smart-merge.sh        # Intelligent file merging
├── settings.json             # Claude Code configuration
└── .mcp.json                # Model Context Protocol server configuration
README.md                     # Copy-and-go setup instructions
```

## Local Todo System Workflow

### **Core Slash Commands**

#### **Planning Commands**
```bash
/project:plan:prd "user authentication system"        # Create comprehensive PRD
/project:plan:feature "dark mode toggle"              # Create focused technical feature  
/project:plan:tasks "prd-001-user-auth"              # Break down into hierarchical tasks
```

#### **Execution Commands**
```bash
/project:do:task "prd-001-user-auth/tasks/1-research" # Execute specific task
/project:current                                      # Show project context
```

### **Context-Aware System**

The system automatically detects project type and adapts:

#### **Project Type Detection**
- **Web Application**: React, Next.js, frontend frameworks → Uses web-app templates
- **API Service**: FastAPI, Django, Express, Go APIs → Uses api-service templates  
- **CLI Tool**: Command-line applications → Uses cli-tool templates
- **SaaS Platform**: Multi-service architecture → Uses saas-platform templates

#### **Hierarchical Task Organization**
```
Tasks use intelligent numbering:
├── 1-research-requirements.md           # Simple sequence
├── 2-backend-implementation.md          
├── 2.1-api-endpoints.md                # Nested breakdown
├── 2.2-data-models.md
├── 3-frontend-development.md
├── 3.1-components.md
├── 3.2-integration.md
└── 4-testing-and-deployment.md
```

### **Complete Workflow Process**
1. **`/project:plan:prd`** - Create comprehensive product requirements with market research
2. **`/project:plan:feature`** - Create focused technical features for specific capabilities
3. **`/project:plan:tasks`** - Break down PRDs/features into implementable tasks with hierarchy
4. **`/project:do:task`** - Execute individual tasks with progress tracking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbusso/claude-workflow](https://github.com/sbusso/claude-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
