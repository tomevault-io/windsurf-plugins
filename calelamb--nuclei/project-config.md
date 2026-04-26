---
trigger: always_on
description: Nuclei is a purpose-built desktop IDE for quantum computing. It combines a Monaco code editor with live circuit visualization, Bloch sphere animations, probability histograms, and an AI teaching assistant called Dirac (powered by Claude API). The primary audience is students taking their first quantum computing course.
---

# CLAUDE.md — Nuclei

## Project Overview

Nuclei is a purpose-built desktop IDE for quantum computing. It combines a Monaco code editor with live circuit visualization, Bloch sphere animations, probability histograms, and an AI teaching assistant called Dirac (powered by Claude API). The primary audience is students taking their first quantum computing course.

**This is a free, open-source tool. No monetization. People download the .dmg and code on it.**

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Desktop Shell | **Tauri 2.x** (Rust) | ~10MB binary, handles process management, file I/O, IPC |
| Frontend | **React 19 + TypeScript** | Vite for bundling, all UI lives here |
| State Management | **Zustand** | Minimal boilerplate, reactive updates across panels |
| Code Editor | **Monaco Editor** | VS Code's engine — syntax highlighting, autocomplete |
| Circuit Rendering | **D3.js** + custom SVG | Live circuit diagrams that update as you type |
| 3D Rendering | **Three.js** | Bloch sphere visualization |
| Charts | **D3.js or Recharts** | Probability histograms |
| Python Kernel | **Managed subprocess + WebSocket** | Executes Qiskit/Cirq/CUDA-Q code |
| AI Assistant | **Claude API** (Haiku for fast, Sonnet for complex) | Dirac — Claude API wrapper with tutor persona, system prompt, context injection, and tool definitions |
| Build | **Vite + Tauri CLI** | HMR in dev, .dmg packaging for release |
| Python Bundling | **conda-pack** (later) | So users don't need Python installed |

## Architecture

Three-layer architecture:

### Layer 1: Tauri Shell (Rust) — `src-tauri/`
- Spawns and manages the Python kernel process
- File system operations (project open/save/watch)
- IPC bridge — typed commands between frontend and backend
- Auto-updater, native menus, system tray

### Layer 2: Frontend (React + TypeScript) — `src/`
- Monaco Editor with Python + Qiskit/Cirq/CUDA-Q aware IntelliSense
- Circuit diagram renderer (D3.js SVG)
- Bloch sphere (Three.js)
- Probability histogram
- Dirac AI chat panel
- Panel layout system (resizable, rearrangeable)
- Zustand stores for circuit state, editor state, simulation results

### Layer 3: Python Kernel — `kernel/`
- WebSocket server that receives code from the frontend
- Framework adapters that convert Qiskit/Cirq/CUDA-Q circuits into a universal CircuitSnapshot format
- Simulation execution and result serialization
- Auto-detects framework from imports

## Key Data Structures

```typescript
// Sent from kernel to frontend on every code change (lightweight, no simulation)
interface CircuitSnapshot {
  framework: 'qiskit' | 'cirq' | 'cuda-q';
  qubit_count: number;
  classical_bit_count: number;
  depth: number;
  gates: Array<{
    type: string;          // 'H', 'CNOT', 'RZ', etc.
    targets: number[];     // qubit indices
    controls: number[];    // control qubit indices
    params: number[];      // rotation angles, etc.
    layer: number;         // depth position
  }>;
}

// Sent from kernel to frontend after explicit execution (Cmd+Enter)
interface SimulationResult {
  state_vector: Array<{ re: number; im: number }>;
  probabilities: Record<string, number>;
  measurements: Record<string, number>;
  bloch_coords: Array<{ x: number; y: number; z: number }>;
  execution_time_ms: number;
}
```

## Data Flow (Critical Path)

1. User types code in Monaco Editor
2. On change (300ms debounce), frontend sends code to Python kernel via WebSocket
3. Kernel parses code, detects framework, builds circuit object
4. Kernel extracts CircuitSnapshot (gate list, qubit count — NO simulation) and returns JSON
5. Frontend renders circuit diagram from snapshot in real time
6. User presses Cmd+Enter → kernel runs full simulation → returns SimulationResult
7. Frontend updates Bloch sphere + histogram panels

## Project Structure

```
nuclei/
├── src-tauri/                   # Rust backend
│   ├── src/
│   │   ├── main.rs              # Entry point, window setup
│   │   ├── commands/            # Tauri IPC commands
│   │   │   ├── kernel.rs        # Python process management
│   │   │   ├── filesystem.rs    # File I/O operations
│   │   │   └── settings.rs      # User preferences
│   │   └── kernel/
│   │       └── manager.rs       # Kernel lifecycle management
│   ├── Cargo.toml
│   └── tauri.conf.json
├── src/                         # React frontend
│   ├── components/
│   │   ├── editor/              # Monaco wrapper
│   │   ├── circuit/             # Circuit diagram renderer
│   │   ├── bloch/               # Bloch sphere (Three.js)
│   │   ├── histogram/           # Probability charts
│   │   ├── dirac/               # AI assistant panel
│   │   ├── terminal/            # Output terminal
│   │   └── layout/              # Panel system & drag-drop
│   ├── stores/                  # Zustand state stores
│   │   ├── circuitStore.ts
│   │   ├── editorStore.ts
│   │   └── simulationStore.ts
│   ├── hooks/                   # Custom React hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calelamb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
