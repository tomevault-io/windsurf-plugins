---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an enterprise-grade data lineage visualization tool built with Vue 3 (frontend) and Python Flask (backend). The application specializes in SQL query lineage analysis with support for both table-level and column-level visualization, featuring intelligent SQL dialect detection (including MySQL backtick syntax), virtualized rendering for large datasets, and key path analysis.

The frontend uses JSPlumb for interactive data flow diagrams, while the backend leverages SQLLineage library for parsing SQL queries and extracting lineage relationships.

This is a frontend-focused refactored version of the original project, with significant performance improvements (4-10x speed increase) while keeping the backend API unchanged.

## Common Development Commands

### Frontend Development
- `npm install` - Install frontend dependencies
- `npm run dev` - Start development server on http://localhost:8620 (with API proxy to port 5000)
- `npm run build` - Build production version to `dist/` directory with relative path support
- `npm run preview` - Preview the built production version

### Backend Development
- `cd api && python -m venv venv` - Create virtual environment
- `venv\Scripts\activate` (Windows) or `source venv/bin/activate` (Linux/Mac) - Activate virtual environment
- `pip install -r requirements.txt` - Install Python dependencies (SQLLineage, Flask, etc.)
- `python api/server.py` - Start backend server on http://localhost:5000

### Docker Deployment
- `./deploy.sh` (Linux/Mac) or `deploy.bat` (Windows) - One-click deployment with health checks
- `docker-compose up --build -d` - Build and start both frontend and backend services
- `docker-compose down` - Stop services
- `docker-compose logs -f` - View logs
- Environment variables available in `.env`: `FRONTEND_PORT`, `BACKEND_PORT`, `CORS_ORIGIN`

### Environment Requirements
- Node.js 22.11.0+
- Python 3.12.9+
- npm 10+
- Docker & Docker Compose (for containerized deployment)

## Core Architecture

### Frontend Architecture (`src/`)

The application follows a single-page application (SPA) pattern with Vue 3 Composition API and modular component architecture:

**Main Entry Point**:
- `src/main.js` - Creates Vue app with router and global configurations

**Core View**:
- `src/views/Index.vue` - The primary application component (5000+ lines) containing:
  - JSPlumb integration for visual connections
  - Virtualized rendering for 2000+ nodes
  - Performance optimizations (hardware acceleration, memory management)
  - Search functionality with 50ms response time
  - Key path analysis algorithms
  - History management with local storage
  - Mini-map navigation system

**Component Structure** (Modular Architecture):
- `src/views/components/SqlInputPanel.vue` - SQL input and analysis controls
- `src/views/components/LineageCanvas.vue` - Main visualization canvas
- `src/views/components/ControlPanel.vue` - Display and export controls
- `src/views/components/TableNode.vue` - Individual table node with field-level details
- `src/views/components/MiniMap.vue` - Navigation overview map with viewport indicator
- `src/views/components/DownloadImage.vue` - Export functionality for high-quality images
- `src/views/components/LoginDialog.vue` - Authentication interface
- `src/views/components/BatchActions.vue` - Batch operation controls

**Configuration Files**:
- `src/views/config/jsplumbConfig.ts` - JSPlumb connection and behavior configuration
- `src/views/config/jsplumbVisualConfig.ts` - Visual styling (colors, stroke widths, animations)
- `src/views/config/tableTypeMappingColor.ts` - Table type color mapping (Source/Middle/Result)

**Methods and Utilities**:
- `src/views/methods/comm.ts` - Core utility methods for JSPlumb operations, drag functionality
- `src/utils/` - General utility functions
- `src/types/index.ts` - TypeScript type definitions
- Uses `panzoom` for canvas zoom/pan functionality with hardware acceleration

**Key Performance Features**:
- Virtualized rendering for large datasets
- Intelligent caching system with memory cleanup
- Batch DOM updates for smooth animations
- Throttled drag operations with alignment guides
- Modular component architecture for better maintainability

### Backend Architecture (`api/`)

**API Server**:
- `server.py` - Flask application with CORS, authentication, and health checks
- Endpoint: `POST /api/lineage` - Accepts SQL query and returns JSON lineage data
- Endpoint: `POST /api/login` - Simple authentication (production should use proper auth)
- Endpoint: `GET /api/health` - Health check for Docker monitoring

**Core Analysis Logic**:
- `lineage.py` - Main API handler that routes requests based on lineage level (table/column)
- `main2.py` - Column-level lineage analysis using SQLLineage with CTE filtering
- `table_lineage.py` - Table-level lineage extraction for high-level overview
- `sql_utils.py` - SQL utility functions for dialect detection and preprocessing

**Key Features**:
- **Dual-mode Analysis**: Table-level for overview, Column-level for detailed field tracking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuzisang/jsplumb-dataLineage-vue-vue3](https://github.com/tuzisang/jsplumb-dataLineage-vue-vue3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
