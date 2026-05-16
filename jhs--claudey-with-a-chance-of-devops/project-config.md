---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claudey With a Chance of DevOps (CWCD) is a DevOps assistant powered by Claude Code. This is currently a conceptual project in early development phase with minimal implementation.

## Project Structure

The repository now contains:
- `README.md` - Project description and whiteboard ideas
- `LICENSE` - Apache License 2.0
- `ui/` - Next.js application with CopilotKit integration
  - Next.js 15.4.4 with React 19.1.0
  - TypeScript and Tailwind CSS
  - CopilotKit sidebar UI for AI-powered DevOps assistance
  - OpenAI adapter for runtime integration
- `chatfield/` - Python implementation of conversational data gathering
  - Complete Python package with decorators for field definitions
  - LLM-powered validation and conversation management using LangGraph
  - Testing suite and example implementations
  - Key components:
    - `Interview` base class for defining data structures
    - `Interviewer` class orchestrating conversations with LangGraph
    - `FieldProxy` string subclass providing transformation access
    - Comprehensive decorator system: `@alice`/`@bob` roles, `@must`/`@reject`/`@hint` validation
    - Type transformations: `@as_int`, `@as_float`, `@as_lang`, etc.
    - Cardinality decorators: `@as_one`, `@as_maybe`, `@as_multi`, `@as_any`
- `chatfield-js/` - TypeScript/JavaScript implementation of Chatfield
  - NPM package @chatfield/core for web applications
  - React integration components and CopilotKit support
  - Comprehensive test suite and usage examples
- `doc/` - Project documentation and diagrams
- `config/` - Container and proxy configuration files
- `docker-compose.yml` - Multi-container development environment

## Architecture: Policy-Driven DevOps

CWCD implements a **policy-driven DevOps** model where organizational standards are enforced, not suggested.

### Core Behavioral Rules

**Apache Answer Policy Supremacy**: When any conflict arises between Claude's general DevOps knowledge and organizational policies stored in Apache Answer, **Apache Answer always wins**. Claude must:
- Escalate to the user rather than override established policies
- Never suggest alternatives that violate organizational policies
- Reference specific policy documents when enforcing requirements

**Progressive Enforcement Model**:
- **New Projects**: Apply full policy enforcement from day-1 initialization
- **Existing Projects**: Provide selective assistance while respecting current state and guiding toward compliance
- **Initialization Discovery**: Use a discovery phase (similar to `/init`) to understand project context and determine applicable policies

### Policy Engine Integration

**Apache Answer as Single Source of Truth**: All organizational DevOps policies are stored in Apache Answer, including:
- Required tech stack (cloud platform, IaC tools, programming languages)  
- Operational standards (backup/restore, disaster recovery, monitoring)
- Compliance requirements and approved patterns
- Historical decisions and their rationale
- Data architecture (dev/prod/staging, bronze/silver/gold data patterns)
- Backend/frontend language/stack specifications
- ADR (Architecture Decision Record) templates and standards

**Knowledge Tagging System**: Apache Answer uses a dual classification:
- **Policy-tagged** knowledge must be obeyed completely - all applicable policies are mandatory
- **Advisory** knowledge provides guidance with community endorsements (thumbs up/down for real-world success/failure)

**Knowledge Evolution**: Apache Answer captures decisions and outcomes as the system operates, building institutional DevOps memory.

**Policy Updates**: Organizations can refine policies either:
- Directly in Apache Answer
- Through conversational alignment with Claude (which then updates Apache Answer)

## Development Context

This project is in the conceptual/planning phase with minimal implementation. Key components under consideration:
- Container support (Docker) - runs in container, can run/pull containers
- Apache Answer integration as policy engine with Policy/Advisory knowledge tagging
- Atomic Agents and Magentic-UI for specialized DevOps personas
- RAGFlow for knowledge aggregation via MCP integration
- Infrastructure as Code enforcement (Terraform)
- Git submodules architecture - minimal monorepo with CLAUDE.md interfacing with team repos
- Claude Code SDK integration with custom system prompts and permission management
- zen-mcp-server for planning support in plan permissions mode
- Product Owner (PO) role designation for users (though actual users likely developers)
- Boot-time configuration discussions for tech stack preferences
- Integration considerations: GitHub, Jira, Confluence (currently out of scope)
- Public registry/KB for community-driven DevOps constitutions

## Current State

The project now includes multiple working implementations:

### UI Application (`/ui`)
- Next.js web application with CopilotKit integration
- Development environment setup with TypeScript and Tailwind CSS
- CopilotSidebar component configured for DevOps assistance
- OpenAI runtime adapter for AI capabilities

Development commands:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhs/Claudey-With-a-Chance-of-DevOps](https://github.com/jhs/Claudey-With-a-Chance-of-DevOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
