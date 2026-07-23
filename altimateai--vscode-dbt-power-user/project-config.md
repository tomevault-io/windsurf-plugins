---
trigger: always_on
description: **vscode-dbt-power-user** is a comprehensive VSCode extension that makes VSCode seamlessly work with dbt (data build tool). It's an open-source project published by Altimate AI that extends VSCode with advanced dbt features including auto-completion, query preview, lineage visualization, documentation generation, and AI-powered features.
---

# Claude Code - dbt Power User VSCode Extension Architecture Guide

## Project Overview

**vscode-dbt-power-user** is a comprehensive VSCode extension that makes VSCode seamlessly work with dbt (data build tool). It's an open-source project published by Altimate AI that extends VSCode with advanced dbt features including auto-completion, query preview, lineage visualization, documentation generation, and AI-powered features.

### Key Statistics

- **Version**: 0.57.3
- **Project Type**: VSCode Extension (TypeScript/React)
- **License**: MIT
- **Architecture**: Multi-layered with webview panels, Python integrations, and MCP server

## High-Level Architecture

### 1. Core Extension Architecture

The extension follows a **dependency injection pattern** using Inversify container:

- **Entry Point**: `src/extension.ts` → `DBTPowerUserExtension`
- **DI Container**: `src/inversify.config.ts` manages all service dependencies
- **Main Extension Class**: `DBTPowerUserExtension` orchestrates all components

### 2. Multi-Process Architecture

The extension operates across multiple processes:

1. **Main Extension Process** (Node.js/TypeScript)

   - VSCode API integration
   - File system operations
   - dbt CLI interactions

2. **Webview Panels** (React/TypeScript)

   - Modern React-based UI components
   - Located in `webview_panels/` directory
   - Built with Vite, uses Antd for UI components

3. **Python Bridge Integration**

   - dbt core/cloud integration via Python scripts
   - Key files: `dbt_core_integration.py`, `dbt_cloud_integration.py`
   - Jupyter kernel for notebook functionality

4. **MCP Server** (Model Context Protocol)
   - AI integration and tool calling functionality
   - Located in `src/mcp/`

### 3. Key Module Organization

```
src/
├── manifest/           # dbt project parsing and management
├── dbt_client/        # dbt integration (core, cloud, fusion)
├── webview_provider/  # Webview panel management
├── autocompletion_provider/ # Language server features
├── services/          # Business logic services
├── commands/          # VSCode command implementations
├── mcp/              # Model Context Protocol server
└── telemetry/        # Analytics and tracking
```

## Core Functionality Areas

### 1. dbt Integration Support

**Multiple Integration Types**:

- **dbt Core**: Direct Python integration via Python bridge
- **dbt Cloud**: API-based integration with dbt Cloud services
- **dbt Fusion**: Command-line integration with dbt-fusion CLI
- **Core Command**: CLI wrapper integration for dbt core

**Key Integration Files**:

- `src/dbt_client/dbtCoreIntegration.ts` - dbt Core Python integration
- `src/dbt_client/dbtCloudIntegration.ts` - dbt Cloud API integration
- `src/dbt_client/dbtFusionCommandIntegration.ts` - dbt Fusion CLI integration
- `dbt_core_integration.py` - Python bridge for Core integration

### 2. Language Server Features

**Provider Architecture**: Each feature implemented as a separate provider:

- `autocompletion_provider/` - IntelliSense for dbt models, macros, sources
- `definition_provider/` - Go-to-definition functionality
- `hover_provider/` - Hover information
- `code_lens_provider/` - Inline actions
- `validation_provider/` - SQL validation

### 3. Webview Panel System

**Modern React Architecture** (`webview_panels/`):

- **Build System**: Vite + TypeScript + React 18
- **State Management**: Redux Toolkit
- **UI Framework**: Antd + custom components
- **Data Visualization**: Perspective.js, Plotly.js

**Key Panels**:

- `modules/dataPilot/` - AI chat interface
- `modules/queryPanel/` - Query results and analysis
- `modules/lineage/` - Data lineage visualization
- `modules/documentationEditor/` - Documentation management
- `modules/insights/` - Project insights and actions

### 4. AI and Advanced Features

**DataPilot AI Integration**:

- Chat-based interface for dbt assistance
- Query explanation and optimization
- Documentation generation
- Test suggestions

**MCP Server Integration**:

- Tool calling for dbt operations
- Integration with Claude and other AI models
- Located in `src/mcp/server.ts`

## Build System and Tooling

### 1. Multi-Stage Build Process

**Main Extension Build** (Webpack):

```bash
npm run webpack        # Development build
npm run vscode:prepublish  # Production build
```

**Webview Panels Build** (Vite):

```bash
npm run panel:webviews  # Build React components
```

### 2. Development Workflow

**Key Scripts**:

- `npm run compile` - Compile the code
- `npm run watch` - Development with hot reload
- `npm run test` - Jest-based testing
- `npm run lint` - ESLint + Prettier
- `npm run build-vsix` - Package extension

**Development Environment**:

- Uses VSCode's built-in debugger ("Launch Extension")
- Hot reload for webview panels
- Python environment auto-detection

### 3. Testing Strategy

**Test Configuration** (`jest.config.js`):

- **Unit Tests**: Jest + ts-jest
- **Mock System**: Custom VSCode API mocks
- **Coverage**: Istanbul-based coverage reporting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AltimateAI/vscode-dbt-power-user](https://github.com/AltimateAI/vscode-dbt-power-user) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
