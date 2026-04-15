---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DAX Catalog v1.0.0** is a production-ready AI-powered Power BI documentation system that transforms raw Power BI DAX metadata into business-friendly documentation using a **7-agent AI pipeline architecture**. The system processes Power BI INFO.VIEW exports and generates comprehensive stakeholder-ready documentation across any business domain with **domain-agnostic intelligence**.

### Key Features
- ✅ **7-Agent Pipeline**: Complete CSV → Domain → Parallel Analysis → Synthesis → Polish workflow
- ✅ **Domain-Agnostic**: Works reliably with any Power BI model without dataset-specific assumptions
- ✅ **Production-Ready**: Clean codebase, v1.0.0, comprehensive error handling and logging
- ✅ **Advanced DAX Linting**: Pattern-based code quality analysis for any DAX formula
- ✅ **Confidence Gating**: Intelligent fallbacks to generic domains when uncertain
- ✅ **Artifact Versioning**: Complete metadata tracking with schema versioning
- ✅ **Web Interface**: React frontend with real-time progress tracking
- ✅ **Stakeholder-Ready Output**: Immediate sharing without additional editing required 

## Essential Commands

### Development & Build
```bash
# Build the project (required for web server)
npm run build

# Development mode CLI
npm run cli <command>

# Run tests
npm run test

# Type checking and linting
npm run typecheck
npm run lint:check
```

### CLI Usage Patterns
```bash
# Full 7-agent pipeline analysis (recommended)
npm run cli orchestrate ./sample-data/test2

# Limit measures for faster testing
npm run cli orchestrate ./sample-data/test2 --max-measures 5

# Legacy single DAX analysis (still supported)
npm run cli analyze 'SUM(Sales[Amount])' --name 'Total Sales'

# File discovery and validation
npm run cli discover ./sample-data/test2 --preview

# Process single CSV (legacy compatibility)
npm run cli process-csv measures.csv --output documentation.md

# Generate sample test data
npm run cli create-samples
```

### Web Application
```bash
# Start both server and client (from web-app directory)
cd web-app && npm run dev

# Individual components
cd web-app && npm run server  # Express server (dynamic port, starts at :3001)
cd web-app/client && npm start  # React client on :3000

# Production mode (build + serve)
cd web-app && npm run build && npm start  # Built React app + server
```

## Architecture Overview

### 7-Agent AI Pipeline Architecture
The core innovation is a sophisticated agent orchestration system that processes Power BI data through specialized AI agents:

**Pipeline Flow:**
```
Agent 0 (CSV Parser) → Agent 1 (Domain Classifier) → [Agents 2,3,4 Parallel] → Agent 5 (Report Synthesis) → Agent 6 (Content Polish)
```

**Agent 0 (CSV Parser)**: Data ingestion and validation
- Parses INFO.VIEW exports with dynamic content detection
- Schema enrichment with format string inference and role detection
- Filters out INFO.VIEW metadata tables (Measures_Table, Column_Table, etc.)
- Builds dynamic ID mappings for relationships

**Agent 1 (Domain Classifier)**: Business context identification with confidence gating
- Identifies industry domain (transportation, finance, compliance, etc.)
- **NEW**: Falls back to "Analytics Model" with generic stakeholders when uncertain
- Determines key stakeholders and regulatory context
- Sets context for subsequent agents

**Agents 2, 3, 4 (Parallel Processing)** with concurrency limiting:
- **Agent 2**: Business glossary and terminology extraction
- **Agent 3**: Data architecture transformation to business processes  
- **Agent 4**: DAX analysis with **NEW** generic DAX linting and business explanations

**Agent 5 (Report Synthesis)**: Unified documentation generation
- Integrates heuristic measure descriptions for complete coverage
- Merges parallel agent outputs into coherent documentation

**Agent 6 (Content Polish & Review)**: Publication-ready content creation
- Comprehensive grammar, style, and readability review
- Professional tone optimization for stakeholders
- Single-page formatted output for immediate sharing
- Quality assurance and consistency validation

### Domain-Agnostic Improvements
- **DAX Linting**: Pattern-based technical analysis (division operators, AVERAGEX issues, etc.)
- **Confidence Gating**: Generic fallbacks prevent hallucination
- **Schema Enrichment**: Smart format detection and role inference
- **No-Fabrication Guardrails**: All agents instructed not to invent domain-specific content
- **Enhanced Logging**: Real-time Claude API call timing and progress visibility

### Key Components

**AgentOrchestrator** (`src/agent-orchestrator.ts`)
- Coordinates the 7-agent pipeline with concurrency limiting (p-limit)
- Manages shared Claude client for API efficiency
- Handles parallel processing of Agents 2, 3, 4
- **NEW**: Artifact versioning with metadata (schema v1.0.0)
- Provides progress callbacks for web UI

**InfoViewParser** (`src/csv-parser.ts`)
- Robust CSV parsing with filename-agnostic detection
- **NEW**: Schema enrichment with format string inference
- Dynamic table/column ID mapping for relationships
- INFO.VIEW metadata filtering to exclude utility tables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kittypony123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
