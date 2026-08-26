---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server for PTC Windchill 13.0.2.x, providing Claude with the ability to interact with Windchill PLM systems. The server implements multiple agents that handle different Windchill domains: parts, documents, changes, workflows, projects, data administration (containers/contexts), and server management (multi-server switching).

## Architecture

### Agent-Based Design
The codebase follows an agent-based architecture where each domain is handled by a specialized agent:

- **BaseAgent** (`src/agents/base-agent.ts`): Abstract base class that provides common MCP server tool registration and error handling

**Core Domain Agents:**
- **PartAgent**: Handles part management, BOM structures, and part searches (24 tools)
- **DocumentAgent**: Comprehensive document management with 25 tools covering:
  - Document lifecycle (create, update, checkout, checkin, revise)
  - Version management (history, iterations, notes)
  - Content management (upload, download, attachments)
  - Relationship management (references, linking)
  - Advanced search (multi-criteria, date ranges, lifecycle states)
  - Bulk operations (batch updates, lifecycle actions)
- **ChangeAgent**: Manages change requests and change processes (16 tools)
- **WorkflowAgent**: Manages workflow items and processes (~12 tools)
- **ProjectAgent**: Handles project-related operations (~10 tools)
- **DataAdminAgent**: Container and context management (13 tools) covering:
  - Container discovery (list products, libraries, organizations, projects, site)
  - Container details and navigation (folders, folder contents)
  - Product/library configuration (option pools, option sets)
  - Uses Windchill REST Services DataAdmin domain (`/DataAdmin/Containers`)
- **ServerManagerAgent**: Multi-server management and switching (5 tools) covering:
  - Server discovery (list all configured servers)
  - Dynamic server switching (change active server at runtime via MCP)
  - Connection testing (verify server availability before switching)
  - Server information retrieval (get current and specific server details)
  - Enables Claude Desktop users to seamlessly switch between Production, Development, and Test environments

**Tier 1 Agents (High Priority - Common Use Cases):**
- **PrincipalMgmtAgent**: User, group, role, and team management (16 tools)
  - User management (list, search, get details, groups, teams)
  - Group management (list, search, members)
  - Team management (list, search, members with roles)
  - Role management (list, assignments)
- **ProdPlatformMgmtAgent**: Options & Variants configuration (12 tools)
  - Option management (list, search, choices)
  - Option set management (list, search, assignments)
  - Choice management
  - Variant expressions and validation
- **NavCriteriaAgent**: BOM navigation and filtering (8 tools)
  - Navigation criteria management
  - Filter expressions and types
  - Applied criteria and defaults by view
- **PartListMgmtAgent**: Parts lists and favorites (10 tools)
  - Part list management (list, search, items)
  - List operations (add, remove, create, delete, update)
  - Sharing and collaboration

**Tier 2 Agents (Module-Specific - Requires Additional Windchill Modules):**
- **ManufacturingAgent**: Manufacturing data (requires Windchill MPMLink) (12 tools)
  - Manufacturing parts management
  - Process plans and operations
  - Resources and work instructions
- **QualityAgent**: Quality management (requires Windchill QMS) (10 tools)
  - Quality inspections
  - Nonconformance reports (NCRs)
  - Corrective actions

**Tier 3 Agents (Specialized Features):**
- **VisualizationAgent**: Creo View visualization services (3 tools)
  - Visualization representations, thumbnails, 3D views
- **EffectivityMgmtAgent**: Date/unit effectivity management (2 tools)
  - Effectivity definitions and effective items
- **CADDocumentMgmtAgent**: CAD-specific document operations (2 tools)
  - CAD documents and structure management
- **ClfStructureAgent**: Classification/taxonomy management (2 tools)
  - Classification nodes and hierarchy
- **SavedSearchAgent**: Saved search management (3 tools)
  - Search definitions and execution
- **ServiceInfoMgmtAgent**: Service information/technical publications (2 tools)
  - Service documentation management
- **PTCAgent**: Common utility entities (2 tools)
  - PTC common entity types and attributes

Each agent extends BaseAgent and defines its own tools with specific input schemas and handlers.

### Windchill Server Management
- **WindchillServerManager** (`src/config/windchill-servers.ts`): Manages multiple Windchill server configurations
- Loads all numbered server configurations from environment variables (`WINDCHILL_*_1`, `WINDCHILL_*_2`, etc.)
- Tracks active server and provides methods to switch between servers
- Supports up to 10 configured servers (can be extended)
- Backward compatible with single-server legacy configuration

### Windchill API Integration
- **WindchillAPIService** (`src/services/windchill-api.ts`): Centralized service for Windchill REST API communication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cars-10/windchill-mcp-server](https://github.com/Cars-10/windchill-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
