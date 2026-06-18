---
trigger: always_on
description: CityAgent is a Cities: Skylines 2 (CS2) mod that integrates Claude AI as an in-game narrative advisor.
---

# CityAgent — CLAUDE.md

## Project Overview

CityAgent is a Cities: Skylines 2 (CS2) mod that integrates Claude AI as an in-game narrative advisor.
Inspired by the storytelling style of YouTuber CityPlannerPlays, it lets players pass live city screenshots
to an LLM along with structured city data, which returns narrative context and build recommendations.

## Architecture

```
CS2 Game (Unity 2022.3.7f1 / DOTS-ECS)
  └── C# Mod (thin bridge layer — src/)
        ├── Reads city stats via ECS systems (population, budget, traffic, zoning)
        ├── Captures screenshots via Unity ScreenCapture API
        └── Calls Claude API over HTTP (async, non-blocking)
              └── Claude API (claude-sonnet-4-6, vision-enabled)
                    ├── Vision input: screenshot (base64 PNG)
                    ├── Tool calls: get_population(), get_budget(), get_traffic(), get_zoning()
                    └── Returns narrative + build recommendations
                          └── Displayed in in-game React/Coherent GT panel (UI/)
```

The C# mod layer stays thin. Intelligence lives in Claude. The React UI is just a web panel embedded in the game.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Mod / game bridge | C# (.NET Standard 2.1, compiled to DLL) |
| Game engine | Unity 2022.3.7f1 (DOTS / ECS architecture) |
| In-game UI | React + TypeScript (Coherent GT — CS2's embedded browser) |
| AI | Claude API (`claude-sonnet-4-6`, multimodal) |
| Distribution | Paradox Mods |

## Project Structure

```
CityAgent/
├── CLAUDE.md
├── README.md
├── .gitignore
├── src/                              # C# mod (compiled to DLL)
│   ├── CityAgent.csproj
│   ├── Mod.cs                        # IMod entry point
│   ├── Settings.cs                   # Mod options (API key, etc.)
│   └── Systems/
│       ├── CityAgentUISystem.cs      # UI bindings / C#↔JS bridge   [Phase 1]
│       ├── CityDataSystem.cs         # Reads ECS city data            [Phase 2]
│       └── ClaudeAPISystem.cs        # Claude API integration         [Phase 3]
└── UI/                               # React UI (Coherent GT)
    ├── package.json
    ├── webpack.config.js
    ├── tsconfig.json
    └── src/
        ├── index.tsx                 # UI entry point / component registration
        └── components/
            ├── CityAgentPanel.tsx    # Main chat panel                [Phase 1]
            ├── ChatMessage.tsx       # Individual message component   [Phase 6]
            └── CityAdvisorButton.tsx # Toolbar trigger button         [Phase 1]
```

## Build Phases

### Phase 1: Toolchain Validation ← CURRENT
**Goal**: A basic C# mod running in CS2 — a button that opens a panel.
- C# mod loads and registers with CS2's mod system
- `CityAgentUISystem` exposes `panelVisible` binding and `togglePanel` trigger
- React panel renders in-game, toggles open/closed
- **Success criteria**: See the panel in-game after a build → deploy cycle

### Phase 2: City Data Reading
**Goal**: Pull live city stats from the ECS and display them in the panel.
- Read population, budget, happiness from CS2 ECS `GameSystemBase` subclasses
- Expose data via `ValueBinding<T>` to the React UI
- Learn the DOTS/ECS access patterns (this is the steepest learning curve)

### Phase 3: Claude API Integration
**Goal**: Send a prompt to Claude and display the response in the panel.
- HTTP client in C# (`System.Net.Http.HttpClient`) calling `api.anthropic.com`
- Async/await to keep the game thread non-blocking
- Display streaming or complete response in the chat panel

### Phase 4: Screenshot Capture
**Goal**: Pass a city screenshot with the prompt (vision input).
- `ScreenCapture.CaptureScreenshot` or render to `RenderTexture`
- Base64-encode the PNG → include in Claude's `messages` array as image content
- Handle file I/O on a background thread

### Phase 5: Agent Tools
**Goal**: Give Claude structured access to live city data.
- Define tool schema: `get_population()`, `get_budget()`, `get_traffic_summary()`, `get_zoning_breakdown()`
- Implement tool_use / tool_result message loop
- Rich context for AI narrative and recommendations

### Phase 6: UI Polish
**Goal**: Production-quality in-game advisor UI.
- Chat history with scroll
- Loading / thinking states
- Narrative text formatting (markdown-ish)
- Settings panel for API key entry (never hardcode)

## Local Setup

### Prerequisites
- Cities: Skylines 2 (set `CS2_INSTALL_PATH` env var to the install directory)
  - Default Steam path: `C:/Program Files (x86)/Steam/steamapps/common/Cities Skylines II`
- Visual Studio 2022 with .NET desktop development workload
- CS2 Mod Template — search "colossal" in VS → Extensions → Manage Extensions
- Node.js 18+ (for UI build)

### Setting CS2_INSTALL_PATH
PowerShell (user-level, persists across sessions):
```powershell
[System.Environment]::SetEnvironmentVariable("CS2_INSTALL_PATH", "C:\Program Files (x86)\Steam\steamapps\common\Cities Skylines II", "User")
```

### Build & Deploy (C# mod)
```bash
# In src/ — close CS2 first!
dotnet build -c Release

# The DLL is output directly to:
# %AppData%/../LocalLow/Colossal Order/Cities Skylines II/Mods/CityAgent/
```

### Build UI
```bash
cd UI
npm install
npm run build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnorris823/CityAgent](https://github.com/dnorris823/CityAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
