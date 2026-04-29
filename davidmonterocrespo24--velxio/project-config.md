---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Velxio** — a fully local, open-source Arduino emulator 
- GitHub: https://github.com/davidmonterocrespo24/velxio
- Frontend: React + Vite + TypeScript with Monaco Editor and visual simulation canvas
- Backend: FastAPI + Python for Arduino code compilation via arduino-cli
- Simulation: Real AVR8 emulation using avr8js with full GPIO/timer/USART support
- Components: Visual electronic components from wokwi-elements (LEDs, resistors, buttons, etc.)
- Auth: Email/password + Google OAuth, JWT in httpOnly cookies
- Project persistence: SQLite via SQLAlchemy 2.0 async + aiosqlite

The project uses **local clones of official Wokwi repositories** in `wokwi-libs/` instead of npm packages.

## Development Commands

### Backend (FastAPI + Python)

**Setup:**
```bash
cd backend
python -m venv venv
venv\Scripts\activate  # Windows
pip install -r requirements.txt
```

**Run development server:**
```bash
cd backend
venv\Scripts\activate
uvicorn app.main:app --reload --port 8001
```

**Access:**
- API: http://localhost:8001
- Docs: http://localhost:8001/docs

### Frontend (React + Vite)

**Setup:**
```bash
cd frontend
npm install
```

**Run development server:**
```bash
cd frontend
npm run dev
```

**Build for production:**
```bash
cd frontend
npm run build
```

**Docker build (skips tsc type-check, uses esbuild only):**
```bash
npm run build:docker
```

**Lint:**
```bash
cd frontend
npm run lint
```

**Access:**
- App: http://localhost:5173

### Wokwi Libraries (Local Repositories)

The project uses local clones of Wokwi repositories in `wokwi-libs/`:
- `wokwi-elements/` - Web Components for electronic parts
- `avr8js/` - AVR8 CPU emulator
- `rp2040js/` - RP2040 emulator

**Update libraries:**
```bash
update-wokwi-libs.bat
```

Or manually:
```bash
cd wokwi-libs/wokwi-elements
git pull origin main
npm install
npm run build
```

### External Dependencies

**arduino-cli** must be installed on your system:
```bash
# Verify installation
arduino-cli version

# Initialize (first time)
arduino-cli core update-index
arduino-cli core install arduino:avr
```

## Architecture

### High-Level Data Flow

1. **Code Editing**: User writes Arduino code → Monaco Editor → Zustand store (`useEditorStore`)
2. **Compilation**: Files → Frontend API call → Backend FastAPI → arduino-cli subprocess → Returns .hex file
3. **Simulation**: .hex file → AVRSimulator.loadHex() → Parsed into Uint16Array → CPU execution loop
4. **Pin Updates**: CPU writes to PORTB/C/D → Port listeners → PinManager → Component state updates
5. **Visual Updates**: Component state changes → React re-renders → wokwi-elements update visually

### Critical Architecture Patterns

**1. Vite Aliases for Local Wokwi Libs**

The `frontend/vite.config.ts` uses path aliases to import from local repositories:
```typescript
resolve: {
  alias: {
    'avr8js': path.resolve(__dirname, '../wokwi-libs/avr8js/dist/esm'),
    '@wokwi/elements': path.resolve(__dirname, '../wokwi-libs/wokwi-elements/dist/esm'),
  },
}
```

**2. Multi-File Workspace (useEditorStore)**

The editor supports multiple files. `useEditorStore` holds:
```typescript
interface WorkspaceFile { id: string; name: string; content: string; modified: boolean; }
// State:
files: WorkspaceFile[]
activeFileId: string
openFileIds: string[]
// Key operations:
createFile, deleteFile, renameFile, setFileContent, markFileSaved,
openFile, closeFile, setActiveFile, loadFiles, setCode (legacy)
```
`setCode` is a legacy setter that writes to the active file's content — used by old call sites.
`loadFiles` replaces all files when loading a saved project.

**3. Multi-File Compilation**

The backend accepts an array of files, not a single code string:
```typescript
// Frontend (compilation.ts)
interface SketchFile { name: string; content: string; }
compileCode(files: SketchFile[], board: string)
// sends: { files, board_fqbn: board }

// Backend (compile.py)
class SketchFile(BaseModel): name: str; content: str
class CompileRequest:
    files: list[SketchFile] | None = None
    code: str | None = None  # legacy fallback
```
The backend promotes the first `.ino` to `sketch.ino` and applies RP2040 Serial redirect only to `sketch.ino`.

**4. AVR Simulation Loop**

The simulation runs at ~60 FPS using `requestAnimationFrame`:
- Each frame executes ~267,000 CPU cycles (16MHz / 60fps)
- Port listeners fire when PORTB/C/D registers change
- PinManager maps Arduino pins to components (e.g., pin 13 → LED_BUILTIN)

**5. State Management with Zustand**

Main stores:
- `useEditorStore`: Multi-file workspace (files[], activeFileId, openFileIds)
- `useSimulatorStore`: Simulation state, components, wires, compiled hex, serialMonitorOpen
- `useAuthStore`: Auth state (persisted in localStorage)
- `useProjectStore`: Current project tracking

**6. Component-Pin Mapping**

Components are connected to Arduino pins via the PinManager:
- PORTB maps to digital pins 8-13 (pin 13 = built-in LED)
- PORTC maps to analog pins A0-A5
- PORTD maps to digital pins 0-7

**7. Wire System**

Wires are stored as objects with start/end endpoints:
```typescript
{
  id: string

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidmonterocrespo24/velxio](https://github.com/davidmonterocrespo24/velxio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
