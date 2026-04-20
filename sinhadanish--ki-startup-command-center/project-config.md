---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Ki's startup command center - a comprehensive development environment for building the world's first Human-AI-Human relationship intelligence platform. This is a lean 3-person startup focused on transforming how couples navigate relationships through AI.

## Development Commands

### Essential Commands (Makefile)
```bash
# Development Commands
make setup     # Complete setup (submodules + Ki platform + automation)
make dev       # Start complete development environment
make ki-setup  # Setup Ki platform specifically  
make stop      # Stop all services
make n8n       # Open n8n automation dashboard (admin/ki2024)
make backup    # Backup everything
make clean     # Clean up everything

# Git Automation Commands (⭐ Claude Code Compatible)
make git-status  # Check status of all submodules and main repository
make git-push    # Push changes in all submodules + main repository
make git-pull    # Pull latest changes from all repositories
```

### Git Automation System
Ki includes comprehensive git automation that handles all submodule repositories automatically. This system is **fully compatible with Claude Code** and eliminates the need for manual submodule management.

**Key Benefits:**
- **Single Command Operations**: One command handles all repositories
- **Claude Code Integration**: Can be executed directly by AI assistants
- **Automated Commit Messages**: Includes timestamps and proper attribution
- **Intelligent Status Reporting**: Shows detailed status across all repositories
- **Error Handling**: Gracefully handles missing submodules and connection issues

### MCP Server Configuration
This project includes comprehensive MCP (Model Context Protocol) server configuration in `.mcp.json` for enhanced Claude Code capabilities:

#### Available MCP Servers
```bash
# View all configured MCP servers for this project
claude mcp list

# Project-scoped MCP servers include:
# - puppeteer: Web automation for competitive analysis and testing
# - filesystem: Enhanced file operations for data room management  
# - github: Repository management across ki-platform, ki-business, ki-automation
# - postgres: Database operations for user analytics and metrics
# - gdrive: Cloud storage integration for investor documents
# - slack: Team communication automation
# - calendar: Meeting scheduling and milestone tracking
# - analytics: Business intelligence and user behavior analysis
```

#### MCP Server Usage
MCP servers are automatically available when working in this project directory and provide:
- **Web Automation**: Competitive analysis, market research, automated testing
- **Database Operations**: User data analysis, metrics tracking, business intelligence
- **File Management**: Advanced document processing, data room organization
- **Team Collaboration**: Automated notifications, meeting scheduling, document sharing
- **Repository Management**: Multi-repo coordination, issue tracking, release management

### Service URLs (when running `make dev`)
- **Ki App**: http://localhost:3001 - Main relationship intelligence platform
- **AI Engine API**: http://localhost:8000 - FastAPI with LangGraph backend
- **Marketing Site**: http://localhost:3000 - Next.js website
- **n8n Automation**: http://localhost:5678 - Business automation workflows

### Testing & Quality (Next Forge Commands)
```bash
# Navigate to Ki platform (Next Forge monorepo)
cd submodules/product/ki-platform

# Core development commands
pnpm dev           # Start all Next.js apps in development mode
pnpm build         # Build all applications for production
pnpm test          # Run Vitest tests across all packages
pnpm lint          # Ultracite linting (Biome-based, faster than ESLint)
pnpm format        # Format code with Ultracite
pnpm typecheck     # TypeScript checking across monorepo
pnpm analyze       # Bundle analysis for optimization
pnpm translate     # Internationalization workflow
pnpm boundaries    # Enforce package boundary rules

# Database operations
pnpm migrate       # Prisma format + generate + push to database

# Package management
pnpm install       # Install dependencies (preferred package manager)
pnpm bump-deps     # Update all dependencies
pnpm bump-ui       # Update shadcn/ui components
pnpm clean         # Clean node_modules across workspace

# Backend (LangGraph AI Engine)
cd apps/langgraph-backend
pytest tests/
python -m pytest tests/ -v
```

### Next Forge Workflow Patterns
```bash
# 1. Start development environment
make dev                    # Starts all services including Next.js apps

# 2. Work on specific applications
cd submodules/product/ki-platform
pnpm dev                   # Development mode for all apps
# Apps run on:
# - Web (marketing): http://localhost:3000
# - App (platform): http://localhost:3001  
# - Docs: http://localhost:3002
# - Storybook: http://localhost:6006

# 3. Add new features following Next Forge patterns
# - Use workspace packages (@repo/*) for shared logic
# - Follow environment variable patterns with keys.ts
# - Implement UI with design-system components
# - Add comprehensive tests with Vitest

# 4. Quality assurance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sinhadanish) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
