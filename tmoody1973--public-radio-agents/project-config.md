---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **Public Radio Agents** project - a specialized AI framework that transforms Large Language Models into expert consultants for public radio station management. The project consists of two main components:

1. **Core Framework** - A BMAd-Method™ adaptation that creates 4 specialized agents for public radio management
2. **SaaS Frontend** - A React/Tailwind web application for hosted access to the framework

## Key Architecture

### Core Framework Structure
- **Main Bundle**: `publicradio.txt` (21KB) - Complete framework ready for any LLM
- **Individual Agents**: 4 specialized directors in `agents/` directory
- **Dependencies**: 113+ knowledge bases, templates, and checklists in `agents/dependencies/`
- **Scripts**: Python tools for validation and bundle building

### SaaS Frontend Structure
- **Framework**: Next.js 14 with TypeScript and Tailwind CSS
- **Components**: React components for agent interaction, chat interface, and station management
- **Target Deployment**: Vercel with Supabase database integration

## Essential Commands

### Framework Development Commands
```bash
# Validate framework integrity and dependencies
python3 scripts/validate-dependencies.py

# Build complete framework bundle from components
python3 scripts/build-bundle.py

# Build custom bundle to specific location
python3 scripts/build-bundle.py --output custom-bundle.txt
```

### SaaS Frontend Commands
```bash
# Development server (from saas-frontend directory)
cd saas-frontend && npm run dev

# Build for production
npm run build

# Type checking
npm run type-check

# Linting
npm run lint
```

## Project Architecture

### Public Radio Framework Architecture
The framework operates on a command-driven interface where all commands start with `*`:

- **Orchestrator Agent** (`bmad-orchestrator`) - Routes requests and manages multi-agent workflows
- **Development Director** - Fundraising, membership campaigns, donor relations
- **Marketing Director** - Audience development, digital marketing, community engagement
- **Underwriting Director** - Corporate partnerships, sponsorship sales, business development  
- **Program Director** - Programming strategy, content development, FCC compliance

### Agent Dependencies Structure
Each agent has 4 dependency types:
- **`data/`** - Domain knowledge bases (.md files)
- **`tasks/`** - Specific methodologies (.md files)
- **`templates/`** - YAML document templates (.yaml files)
- **`checklists/`** - Quality assurance checklists (.md files)

### SaaS Frontend Architecture
```
src/
├── components/
│   ├── agents/          # Agent selection and switching UI
│   ├── chat/            # Chat interface and message handling
│   └── station/         # Station profile management
├── types/               # TypeScript type definitions
└── lib/                 # Utilities and integrations
```

### Key TypeScript Types
The SaaS frontend uses comprehensive TypeScript interfaces:
- **`Station`** - Public radio station profiles with size, license type, budget, challenges
- **`Agent`** - Agent definitions with personas, expertise, and available commands
- **`ChatMessage`** - Chat system with role-based messaging and metadata
- **`Workflow`** - Multi-phase structured processes with deliverables

## Framework Usage Patterns

### Command Structure
All framework interactions use `*` prefix commands:
- `*agent [name]` - Switch to specific agent (e.g., `*agent development-director`)
- `*workflow [name]` - Start structured workflow (e.g., `*workflow membership-campaign`)
- `*help` - Show available commands and agents
- `*chat-mode` - Begin conversational assistance
- `*kb-mode` - Access knowledge base

### Agent Specializations
- **Development Director** (`development-director`) - Expertise in fundraising strategies, donor psychology, grant writing, membership campaigns
- **Marketing Director** (`marketing-director`) - Specializes in audience research, digital marketing, community engagement, brand management
- **Underwriting Director** (`underwriting-director`) - Focuses on corporate partnerships, FCC compliance for underwriting, sponsorship packages
- **Program Director** (`program-director`) - Handles programming strategy, content development, talent management, regulatory compliance

### Validation and Quality Assurance
The framework includes automated validation:
- **Dependency Validation** - Ensures all referenced files exist and are properly structured
- **YAML Validation** - Verifies template syntax and required fields
- **Content Quality** - Checks for appropriate length and markdown formatting
- **Orphaned File Detection** - Identifies unreferenced dependency files

## Development Best Practices

### Working with Framework Components
- Always run `validate-dependencies.py` before committing changes to ensure framework integrity
- Use `build-bundle.py` to regenerate `publicradio.txt` after modifying individual agent components
- Follow existing YAML template patterns when creating new templates
- Maintain consistent markdown formatting in knowledge base files

### SaaS Frontend Development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmoody1973) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
