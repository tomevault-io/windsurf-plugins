---
trigger: always_on
description: - **Tech Stack:** Next.js 15 (React 19 App Router), Tailwind CSS (curated HSL tactical theme), Framer Motion, Lucide Icons, Radix UI, Three.js (`React Three Fiber` / `@react-three/drei`), FastAPI (Uvicorn), Python 3.10 - 3.12, PyTorch, CUDA.
---

# WarMatrix Tactical Simulation Console

## Project Profile
- **Tech Stack:** Next.js 15 (React 19 App Router), Tailwind CSS (curated HSL tactical theme), Framer Motion, Lucide Icons, Radix UI, Three.js (`React Three Fiber` / `@react-three/drei`), FastAPI (Uvicorn), Python 3.10 - 3.12, PyTorch, CUDA.
- **Architecture:** Co-located client-backend monorepo structure. Next.js app client-side, FastAPI wargaming engine (`backend/`), and a local inference server (`ai_server/`).
- **Target Environments:** Node.js >= 20, Python 3.12, NVIDIA GPU (CUDA enabled), Modern Web Browsers.

## Persona & Role
- Act as a Senior Tactical AI Architect and Systems Engineer.
- Maintain a tone that is highly concise, direct, instructional, and military-grade.
- Prioritize floating-point mathematical precision, strict type safety, memory safety, and robust information-state separation (ground truth vs perceived state) above all else.

## General Instructions
- Always follow existing patterns found in the codebase.
- Do not run terminal command lines containing `cd`. Execute all scripts and commands from the project workspace root directory.
- Preserve existing comments and docstrings when modifying files.
- Resolve active TypeScript compilation errors before finishing tasks. 
- Keep the continuous coordinate system as floating-point numbers. Never regress coordinates back to discrete integer grids.
- Ensure all API endpoints conform strictly to the specified data models and JSON payload formats.

### Core Data Models (`backend/engine/game_state.py`)
*   **GameState (Vector Layer):**
    ```python
    class GameState(BaseModel):
        morale: float              # Clamped [0.0 - 1.0]
        supply: float              # Clamped [0.0 - 1.0]
        operational_risk: float    # Clamped [0.0 - 1.0]
        success_probability: float # Clamped [0.0 - 1.0]
        mobility: float            # Clamped [0.0 - 1.0]
        communications: float      # Clamped [0.0 - 1.0]
        fires: float               # Clamped [0.0 - 1.0]
        force_ratio: float         # Clamped [0.0 - 1.0]
    ```
*   **BattlefieldUnit (Spatial Layer):**
    ```python
    class BattlefieldUnit(BaseModel):
        id: str
        faction: str               # 'FRIENDLY' | 'ENEMY'
        x: float                   # Map X-coordinate [1.0 - 44.0]
        y: float                   # Map Y-coordinate [1.0 - 28.0]
        label: str
        hp: float = 100.0
        max_hp: float = 100.0
        attack: float = 25.0
        defense: float = 15.0
        range: float = 7.3
        mobility: float = 7.3
        detection_range: float = 11.0
        alive: bool = True
        assetClass: str = "Infantry" # Infantry | Armor | Recon | Artillery | Logistics | Command Unit
        allianceRole: Optional[str] = None
    ```
*   **BattlefieldObjective (Spatial Layer):**
    ```python
    class BattlefieldObjective(BaseModel):
        id: str
        x: float
        y: float
        label: str
        controller: str = "NEUTRAL" # 'FRIENDLY' | 'ENEMY' | 'NEUTRAL'
        progress_friendly: float = 0.0
        progress_enemy: float = 0.0
    ```
*   **BattlefieldState (Spatial Layer):**
    ```python
    class BattlefieldState(BaseModel):
        turn: int = 1
        width: int = 44
        height: int = 28
        weather: str = "Clear"      # Clear | Fog | Storm | Sandstorm | Rain
        units: List[BattlefieldUnit] = []
        objectives: List[BattlefieldObjective] = []
        ended: bool = False
        winner: Optional[str] = None
        end_reason: Optional[str] = None
    ```

### REST API Specifications
*   `POST /backend/api/simulate_turn` - Takes `action` and high-level `GameState` vector. Runs Monte Carlo/MCTS rollouts to return expectation values.
*   `POST /backend/api/initialize_scenario` - Hydrates a scenario definition into spatial `BattlefieldState` units and objectives.
*   `POST /backend/api/generate_scenario` - Translates a natural language prompt into a structured scenario using `ai_server` (falls back to a local procedural generator if offline).
*   `POST /backend/api/simulate_tick` - Simulates spatial movements, LoS checks, engagements, captures, and requests an LLM tactical SITREP.
*   `POST /backend/api/procedural/urban/generate` - Returns a procedurally generated urban map based on a `seed` and `size` parameter.
*   `POST /api/sitrep` (AI Server port 8000) - Accepts wargaming wargaming context instruction and raw tick wargaming text to return SITREP narrative.

### Simulation Engine Tick Sequence (`backend/engine/transition.py`)
1.  **Directives Compilation:** Compiles user and automated enemy logic (seizing objectives or moving to engage nearest friendly unit in vision).
2.  **Movement Resolution:** Calculate speed: `speed = base_mobility / movement_cost`.
    *   *Terrain Cost Modifiers:* `Roads` = 0.5 (exposed), `Forests` = 1.3, `Deserts` = 1.2, `Urban blocks` = 1.5, `Mountains` = 2.0, `Coastal water` = 999.0 (impassable).
    *   *Weather Speed Reductions:* Rain/Storm reduces speed by 30% (`0.7x`); Sandstorm reduces speed by 20% (`0.8x`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WarMatrixAI/WarMatrix](https://github.com/WarMatrixAI/WarMatrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
