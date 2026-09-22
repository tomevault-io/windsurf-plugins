---
trigger: always_on
description: 3DTD - Standalone Tower Defense auf Google Photorealistic 3D Tiles (über Cesium Ion oder die Google Maps API)
---

# CLAUDE.md - 3DTD

## Projekt

3DTD - Standalone Tower Defense auf Google Photorealistic 3D Tiles (über Cesium Ion oder die Google Maps API)

## Befehle

```bash
npm start       # Development Server (http://localhost:4200)
npm run build   # Production Build
npm test        # vitest
npm run lint
```

## Architektur

- Angular 22 Standalone Components (nur UI)
- Three.js + 3DTilesRendererJS für 3D-Rendering
- **Event-driven Game Engine** - Manager kommunizieren via GameEventBus
- **Signal Store** - 6 Sub-Stores als Single Source of Truth (Game, UI, Engine, Location, Research, Debug)
- Kein Backend im Spiel-Client - komplett clientseitig (Python-Backend nur für AI-Training)
- **Wave-Director ist regelbasiert** (`ai/core/rule-director.ts` + `gate-controller.ts`),
  läuft ohne Server, ohne Modell, ohne ONNX-Runtime. Das ONNX-Modell ist Opt-in
  im Debug-Fenster - Begründung in [AI_WAVE_DIRECTOR_PLAN.md](docs/AI_WAVE_DIRECTOR_PLAN.md)
- Tile-Zugang: Cesium-Ion-Token (Standard) oder Google-Maps-Key. `ConfigService` liest ihn aus drei Quellen, die
  spätere gewinnt: `environment.ts` (Vorlage `environment.template.ts`), `public/runtime-config.json`, Token-Dialog
  (localStorage `3dtd-tile-credentials`)

## Projektstruktur

```
src/app/
├── app.ts                      # Root Component (AppComponent)
├── app.config.ts               # Provider Config
├── app.routes.ts               # Routing
├── tower-defense.component.*   # Haupt-Spielkomponente (.ts, .html, .scss)
├── ai/                         # AI System (Browser)
│   ├── core/                   # Regel-Director, Gate-Controller, Templates, State-Encoder, Decision-Explainer
│   └── training/               # Bot System (Strategy Pattern), Training-Session, WebSocket-Client
│       ├── bots/               # StrategyBot, Factory
│       └── strategies/         # Placement, Upgrade, Wave, Research, Ability Strategies
├── game-engine/                # Event Bus, VFX/Audio/BackgroundMusic/ScreenShake Services (Three.js-coupled, Angular-frei)
├── components/                 # UI Components (compass, game-header, game-sidebar, etc.)
├── configs/                    # Tower/Enemy/Projectile/Combat/Research/Audio + Wave-Curriculum-Configs
├── core/                       # GameObject/Component-Basis, ConfigService
├── devworld/                   # DevWorld Offline-Entwicklungsumgebung
├── entities/                   # Enemy, Tower, Projectile (+ tower-targeting.util, enemy-rush)
├── game-components/            # ECS Components (transform, health, movement, combat, etc.)
├── integration/                # Cross-System Integration-Tests
├── interfaces/                 # Provider-Interfaces (StreetNetwork, Terrain)
├── managers/                   # Manager (Enemy, Tower, Wave, Research, Ability, Hero usw., event-driven), game-state/ (Ledger, Lifecycle, Clock), worm/, audio/ (Spatial Audio)
├── models/                     # Type Definitions (game.types, location.types, status-effects)
├── replay/                     # Replay der letzten Welle: Recorder, Player (docs/REPLAY.md)
├── services/                   # Angular Services (Subfolders: combat/, debug/, facade/, infrastructure/, location/, onboarding/, world/)
├── store/                      # Signal Stores (Game, UI, Engine, Location, Research, Debug)
├── styles/                     # Theme-Tokens (td-theme.ts)
├── three-engine/               # 3D Rendering: Engine, CameraRig, Tiles, renderers/ (inkl. Shader), post-processing/
├── utils/                      # Shared Utilities (geo-utils, damage-calculator, global-route-grid, route-corridor)
└── workers/                    # Web Workers (Pathfinding, Heartbeat)

training-backend/               # Python Training Backend (nur für Trainingsläufe)
├── server.py                   # WebSocket Server (:3001), Decoder, A/B-Verteilung
├── manage_server.py            # Start/Stop/Status als Hintergrundprozess
├── directors.py                # Austauschbare Wave-Designer (model/rules/random/maxgate)
├── schema.py                   # Lädt generated/ai-schema.json (Templates, Curriculum, Masken)
├── config.py                   # Hyperparameter, DIRECTOR_ROSTER
├── core/
│   ├── model.py                # Neural Network (Conv1D + Dense, State 208 → 36 Outputs)
│   ├── trainer.py              # PPO Training Algorithm
│   └── reward.py               # Reward Function (4 Terms: death, drama, pacing, swarm_size)
├── utils/logger.py             # Console + JSONL-Logging
├── dashboard/                  # Web Dashboard (:3002)
│   ├── app.py                  # FastAPI Server
│   └── static/                 # Chart.js UI
├── generated/ai-schema.json    # Aus den TS-Configs generiert (`npm run ai-schema`)
├── scripts/                    # ONNX-Export, Log-Analyse, Training-Inspector
├── tests/                      # pytest (Schema, Encoder, Reward, Directors, Gate-Loop, Training-Log)
├── requirements.txt            # Python-Abhängigkeiten
├── start.bat / start.sh        # Start-Skripte (Windows, Unix)
├── checkpoints/                # Model Checkpoints (+ archive-<datum>/)
└── docs/                       # Backend-Dokumentation
```

## Wichtig

- **Kein `npm start` ohne Befehl**
- **Keine Commits ohne Befehl**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingel81/3dtd](https://github.com/ingel81/3dtd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
