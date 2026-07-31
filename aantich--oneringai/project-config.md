---
trigger: always_on
description: This document provides context for AI assistants to continue development of the AMOS application.
---

# AMOS Development Guide

This document provides context for AI assistants to continue development of the AMOS application.

## Project Overview

**Name**: AMOS (Advanced Multimodal Orchestration System)
**Purpose**: Terminal-based agentic chat application with runtime configuration
**Built on**: `@everworker/oneringai` library (UniversalAgent)
**Language**: TypeScript (strict mode)
**Runtime**: Node.js 18+
**Package Type**: ESM

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         AmosApp                                  │
│  Main application class - ties all components together          │
└────────────────┬────────────────────────────────────────────────┘
                 │
    ┌────────────┼────────────┬────────────┬────────────┬────────────┐
    │            │            │            │            │            │
    ▼            ▼            ▼            ▼            ▼            ▼
┌────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│Terminal│ │Command   │ │Connector │ │Tool      │ │Prompt    │ │Agent     │
│   UI   │ │Processor │ │Manager   │ │Loader    │ │Manager   │ │Runner    │
└────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
    │            │            │            │            │            │
    │            │            │            │            │            ▼
    │            │            │            │            │     ┌──────────────┐
    │            │            │            │            │     │Universal     │
    │            │            │            │            │     │Agent         │
    │            │            │            │            │     │(@everworker)  │
    │            │            │            │            │     └──────────────┘
    ▼            ▼            ▼            ▼            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      data/ (filesystem)                          │
│  config.json | connectors/*.json | sessions/ | tools/*.js       │
│  prompts/*.md                                                    │
└─────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
apps/amos/
├── src/
│   ├── index.ts                    # Entry point, signal handlers
│   ├── app.ts                      # AmosApp - main orchestrator
│   │
│   ├── config/
│   │   ├── types.ts                # All type definitions + DEFAULT_CONFIG
│   │   ├── ConfigManager.ts        # Load/save config to JSON
│   │   └── index.ts
│   │
│   ├── commands/
│   │   ├── CommandProcessor.ts     # Command routing, parsing, execution
│   │   ├── BaseCommand.ts          # Abstract base class for commands
│   │   ├── index.ts
│   │   └── commands/
│   │       ├── HelpCommand.ts      # /help
│   │       ├── ModelCommand.ts     # /model - uses MODEL_REGISTRY
│   │       ├── VendorCommand.ts    # /vendor - uses Vendor enum
│   │       ├── ConnectorCommand.ts # /connector add|edit|delete|generate|use
│   │       ├── ToolCommand.ts      # /tool list|enable|disable|reload
│   │       ├── PromptCommand.ts    # /prompt list|show|use|clear|create|edit|delete
│   │       ├── SessionCommand.ts   # /session save|load|list|new
│   │       ├── ConfigCommand.ts    # /config get|set|reset
│   │       ├── UtilCommands.ts     # /clear, /exit, /status, /history
│   │       └── index.ts
│   │
│   ├── connectors/
│   │   ├── ConnectorManager.ts     # CRUD + Connector.create() registration
│   │   └── index.ts
│   │
│   ├── tools/
│   │   ├── ToolLoader.ts           # Built-in + custom tool loading
│   │   └── index.ts
│   │
│   ├── prompts/
│   │   ├── PromptManager.ts        # Prompt template management
│   │   └── index.ts
│   │
│   ├── agent/
│   │   ├── AgentRunner.ts          # UniversalAgent wrapper
│   │   └── index.ts
│   │
│   └── ui/
│       ├── Terminal.ts             # readline, chalk, prompts, spinners
│       └── index.ts
│
├── data/
│   ├── config.json                 # App configuration (created on first run)
│   ├── connectors/                 # Connector JSON files
│   ├── sessions/                   # Session persistence
│   ├── tools/                      # Custom tools (.js files)
│   │   └── example-tool.js
│   ├── prompts/                    # System prompt templates (.md files)
│   │   ├── default.md              # Default helpful assistant
│   │   ├── coding-assistant.md     # Expert coding assistant (basic)
│   │   ├── coding-agent.md         # Autonomous coding agent with full tools
│   │   ├── research-analyst.md     # Research and analysis
│   │   └── writing-editor.md       # Writing and editing
│   └── logs/                       # Log files (dev mode)
│
├── package.json
├── tsconfig.json
├── .gitignore
├── README.md
└── CLAUDE.md                       # This file
```

## Key Components

### 1. AmosApp (`src/app.ts`)

Main orchestrator implementing `IAmosApp` interface:

```typescript
interface IAmosApp {
  // Configuration
  getConfig(): AmosConfig;
  updateConfig(partial: Partial<AmosConfig>): void;
  saveConfig(): Promise<void>;

  // Component access
  getConnectorManager(): IConnectorManager;
  getToolLoader(): IToolLoader;
  getPromptManager(): IPromptManager;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aantich/oneringai](https://github.com/aantich/oneringai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
