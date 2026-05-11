---
trigger: always_on
description: - [Repository Purpose](#repository-purpose)
---

# AGENTS.md - Repository Context for AI Assistants

## 📑 Table of Contents

- [Repository Purpose](#repository-purpose)
- [What This Repo Contains](#what-this-repo-contains)
- [What This Repo Does NOT Contain](#what-this-repo-does-not-contain)
- [Repository Structure](#repository-structure)
- [Architecture Principles](#architecture-principles)
- [Key Technologies](#key-technologies)
- [AI Assistant Guidelines](#ai-assistant-guidelines-when-working-with-this-repo)
- [GitHub Copilot Coding Agent Guidelines](#github-copilot-coding-agent-guidelines)
  - [TypeScript Development Patterns](#typescript-development-patterns)
  - [MCP Tool Implementation Patterns](#mcp-tool-implementation-patterns)
  - [Layer Service Integration Patterns](#layer-service-integration-patterns)
  - [Service Architecture Patterns](#service-architecture-patterns)
  - [Testing Requirements](#testing-requirements)
  - [File Organization Rules](#file-organization-rules)
  - [Build and Development Workflow](#build-and-development-workflow)
  - [Submodule Management](#submodule-management)
  - [Performance Guidelines](#performance-guidelines)
  - [Common Anti-Patterns to Avoid](#common-anti-patterns-to-avoid)
- [Hard-Learned Lessons](#hard-learned-lessons)
- [Common Development Tasks](#common-development-tasks)
- [GitHub Copilot Development Workflows](#github-copilot-development-workflows)
  - [New MCP Tool Creation Workflow](#new-mcp-tool-creation-workflow)
  - [Layer System Extension Workflow](#layer-system-extension-workflow)
  - [Service Enhancement Workflow](#service-enhancement-workflow)
  - [Quality Assurance Patterns](#quality-assurance-patterns)
- [Release Process & Publishing Workflow](#release-process--publishing-workflow)
  - [Pre-Release Checklist](#pre-release-checklist)
  - [Publishing to NPM](#publishing-to-npm)
  - [GitHub Release](#github-release)
  - [Post-Release Verification](#post-release-verification)
  - [Rollback Procedure](#rollback-procedure-if-needed)
  - [Common Release Issues & Solutions](#common-release-issues--solutions)
  - [Quick Release Command Reference](#quick-release-command-reference)
- [Integration Points](#integration-points)

---

## Repository Purpose

This is the **MCP server implementation** for the BC Code Intelligence (bc-code-intel) system.

## What This Repo Contains

- **TypeScript MCP Server**: Full Model Context Protocol implementation
- **Layer Resolution System**: Multi-source knowledge with intelligent override system
- **MCP Tools**: 20+ tools providing BC knowledge access, specialist discovery, and handoff management
- **Service Architecture**: Modular services for knowledge, methodologies, code analysis, and specialist management
- **Build System**: Complete TypeScript build pipeline with testing

## What This Repo Does NOT Contain

- **Knowledge Content**: Actual BC knowledge is in bc-code-intelligence repo (linked via submodule)
- **Markdown Files**: No knowledge content directly in this repository
- **Domain Expertise**: Business logic focuses on layer resolution, not BC domain knowledge

## Repository Structure

```
src/
├── layers/                 # Layer resolution system
│   ├── layer-service.ts   # Core layer management and resolution
│   ├── embedded-layer.ts  # Reads from embedded-knowledge/ submodule
│   ├── project-layer.ts   # Local ./bc-code-intel-overrides/ detection
│   └── git-layer.ts       # Future: Git repository layers
├── services/              # MCP tools and business logic
│   ├── knowledge-service.ts    # Knowledge discovery and retrieval
│   ├── methodology-service.ts  # Workflow and systematic analysis
│   └── code-analysis-service.ts # BC code pattern analysis
├── types/                 # TypeScript interfaces and types
│   ├── bc-knowledge.ts    # Knowledge content interfaces
│   ├── layer-types.ts     # Layer system types
│   └── mcp-types.ts       # MCP protocol extensions
└── index.ts              # MCP server entry point

embedded-knowledge/        # Git submodule → bc-code-intelligence
package.json              # Dependencies and build scripts
tsconfig.json            # TypeScript configuration
```

## Architecture Principles

- **Clean Separation**: Server implementation vs knowledge content via submodule
- **Layer-Based Resolution**: Embedded → Company → Team → Project override system
- **Zero-Config Experience**: Works immediately with embedded knowledge via submodule
- **Extensible Design**: Foundation ready for enterprise customization layers
- **Version Awareness**: BC version compatibility filtering throughout
- **Protocol Compliance**: Full MCP specification implementation

## Key Technologies

- **Model Context Protocol (MCP)**: Primary interface for AI tool integration
- **TypeScript**: Strict typing throughout for reliability
- **Git Submodules**: Clean knowledge content integration
- **Fuse.js**: Intelligent knowledge search and discovery
- **YAML**: Frontmatter parsing for knowledge metadata

## AI Assistant Guidelines When Working With This Repo

1. **Code Quality**: Maintain TypeScript strict mode compliance
2. **MCP Compatibility**: Preserve protocol compatibility in all tool implementations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeremyVyska/bc-code-intelligence-mcp](https://github.com/JeremyVyska/bc-code-intelligence-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
