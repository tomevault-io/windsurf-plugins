---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LogicDrawer is a web-based interactive digital logic circuit designer and simulator with advanced AI-powered features. **Version 1.1.0** (actively developed).

**Core Components**:

- **Frontend**: TypeScript/Vite-based canvas application for visual circuit design and simulation
- **Backend**: Node.js/Express server with MongoDB for user management, circuit storage, and API services
- **AI/ML Stack**:
  - Circuit detection via Python/YOLO v8 model with PyTorch
  - AI assistant powered by Google Gemini 2.5 Flash (with Mistral API as alternative)
  - Tool-based agent system for circuit manipulation and analysis
  - Detection statistics tracking for usage analytics

## Development Commands

### Frontend Development

```bash
npm run dev              # Start frontend dev server on port 4000
npm run build            # Build frontend (TypeScript + Vite)
npm run preview          # Preview production build
npm run lint             # Run ESLint with auto-fix + type checking
npm run format           # Format code with Prettier
npm run format:check     # Check formatting without changes
```

### Backend Development

```bash
npm run dev:server       # Start backend dev server (ts-node-dev)
cd server && npm run dev # Alternative: run from server directory
cd server && npm run build # Build backend TypeScript
```

### Combined Development

```bash
npm run dev:all          # Run frontend + backend concurrently
npm run dev:network      # Same but expose frontend to network
npm run build:all        # Build both frontend and backend
```

### Testing

```bash
npm test                        # Run all tests with Vitest
npm run test:detection          # Run YOLO circuit detection tests (requires Python venv)
                                # Uses: ./server/venv/bin/python
```

### Python Environment (AI Features)

```bash
cd server
python3 -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
```

### Environment Setup

Copy `server/.env.example` to `server/.env` and configure:

- `MONGODB_URI`: MongoDB connection string (local or cloud)
- `GOOGLE_API_KEY`: For Gemini 2.5 Flash AI features
- `JWT_SECRET`: Authentication secret (use strong random string)
- `PORT`: Server port (default: 3000)
- `LOGICDRAWER_DEV`: Set to `"true"` to enable console logging (dev mode)

## Architecture

### Core Design Pattern

LogicDrawer uses an **object-oriented component-based architecture** where all circuit elements inherit from a base `Component` class:

1. **Component Hierarchy**
   - `Component` (abstract base class): Defines position, size, ports (inputs/outputs), rotation
   - `LogicGate`: Base class for all logic gates (AND, OR, NOT, XOR, etc.)
   - Specialized gates in `src/models/gates/`: Basic gates, multiplexers, adders, subtractors, decoders
   - Sequential elements in `src/models/Sequential/`: D-latches, D flip-flops
   - I/O components in `src/models/components/`: Switches, buttons, LEDs, displays, clock generators

2. **Wire and Port System**
   - `Wire` class connects `Port` objects between components
   - Ports have `type` (input/output), `bitWidth`, `value` (boolean or BitArray)
   - Wires support control points for routing, multi-bit values
   - Bit width validation ensures compatible connections

3. **CircuitBoard (Main Controller)**
   - Located at `src/models/CircuitBoard.ts` (~2700 lines)
   - Manages all components and wires in the circuit
   - Handles canvas rendering, zoom/pan, minimap
   - Implements simulation engine (signal propagation)
   - Manages user interactions: drag-drop, selection, wire routing
   - Coordinates with utility classes:
     - `ActionHistory`: Undo/redo functionality
     - `TruthTableManager`: Generate truth tables from circuits
     - `KarnaughMap`: K-map generation and analysis
     - `GatePanel`: Properties panel for gate configuration

4. **Main Application Entry**
   - `src/main.ts` (~2335 lines): Application initialization, event handlers, UI controllers
   - Sets up CircuitBoard, AIAgent, authentication, repository
   - Manages toolbar, gate panel, AI chat interface
   - Handles file import/export (JSON, Verilog, PNG)

### AI Integration

1. **AIAgent System** (`src/ai/AIAgent.ts`)
   - **Tool-based architecture** with specialized tools located in `src/ai/tools/`:
     - `VerilogImportTool`: Parse and import Verilog HDL code
     - `CircuitDetectionTool`: Detect circuits from hand-drawn images using YOLO
     - `ImageAnalysisTool`: Analyze circuit images using vision models
     - `TruthTableImageTool`: Extract truth tables from images via OCR
     - `KMapImageTool`: Extract and analyze Karnaugh maps from images
     - `AddComponentsTool`: Programmatically add components to circuit board
     - `ConnectComponentsTool`: Auto-connect components with intelligent port selection
     - `GetCircuitSummaryTool`: Get JSON summary of current circuit state
     - `FinalAnswerTool`: Format final responses to users
   - Uses **Google Gemini 2.5 Flash** model with streaming responses
   - Message queue system for conversation history management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaanAydinli/LogicDrawer](https://github.com/KaanAydinli/LogicDrawer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
