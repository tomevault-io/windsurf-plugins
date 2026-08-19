---
trigger: always_on
description: Reglas generales del repositorio para **cualquier agente de programación** (Claude Code, Codex, Copilot, u otro). Independiente de herramienta.
---

# AGENTS.md — CyberCBA 2077

Reglas generales del repositorio para **cualquier agente de programación** (Claude Code, Codex, Copilot, u otro). Independiente de herramienta.

> Nota de precisión: "Exodus Systems" aparece únicamente como texto de sabor diegético en el Splash y como crédito en el Menú (`src/app/GameApp.cpp`, pantallas Splash/Menu — "AN EXODUS SYSTEMS PRODUCTION"). No hay ningún sistema, clase ni entidad narrativa con ese nombre en el código o los docs; no lo trates como un módulo o mecánica real.

## 1. Qué es CyberCBA 2077

RPG narrativo por turnos en C++17/raylib: exploración por nodos, investigación, diálogo y encuentros por turnos — ver `docs/campaign-bible.md` y ADR `0008-narrative-rpg-pivot.md`. Versión actual: **v0.1.0 — "La última transmisión"** (ver `vcpkg.json`, `README.md`). El prototipo top-down original (movimiento libre, colisión) fue reemplazado en `Screen::World` por un mapa de nodos real (ADR `0014-node-based-world-screen.md`); `WorldLayout` se preserva solo como fondo ilustrado procedural, sin colisión ni movimiento.

- Escenario: Neo-Córdoba 2077, "La ciudad que nos olvidó".
- El jugador elige entre **Emmanuel "Emma" Vázquez** (intrusión de enlace / hacking) o **Magalí "Magga" Temerro** (impacto táctico / combate). La elección cambia quién responde la transmisión, la ruta del prólogo y el encuentro por turnos que resuelve el obstáculo (`Screen::Encounter`); ambas rutas convergen en la entrada clausurada del Neometro.
- **Producto jugable vs. capa académica**: la capa académica (estructuras de datos, tests, benchmarks) queda fuera de la ficción y **no aparece en la UI del jugador**. `QueueChallenge` (`include/cybercba/QueueChallenge.hpp`, `src/domain/QueueChallenge.cpp`) es un dominio educativo de referencia — no es contenido de la campaña v0.1.0 (ver `docs/educational-design.md`, ADR `0003-domain-boundary.md`).
- Objetivo de v0.1.0: prólogo jugable completo (selección de personaje → intro → mapa de nodos del refugio → investigación → apagón → recuperación de energía → transmisión fragmentada → encuentro por turnos → elección narrativa → ruta exterior → control de seguridad → entrada Neometro → teaser de la próxima versión). v0.2.0 ("Neometro: El último convoy") es **planeado**, no implementado — incluirá las estructuras Queue/Stack académicas sin STL (`docs/roadmap.md`).

## 2. Regla de lectura previa

Antes de modificar código, todo agente debe leer:

- Este `AGENTS.md`.
- `README.md`.
- Los Markdown relevantes en `docs/` para el módulo afectado (ver tabla en §3).
- Los ADRs en `docs/adrs/` (`0001`–`0006`).
- Configuración y tests relacionados con el cambio.

Si el cambio altera arquitectura, comportamiento, controles, assets, formato de datos, comandos de build o flujo de misión, **la documentación correspondiente debe actualizarse en el mismo cambio**.

Documentos con contenido **desactualizado, marcado explícitamente como histórico, o scratch** — no tratarlos como fuente vigente sin verificar contra el código:

| Documento | Estado |
|---|---|
| `docs/ARQUITECTURA_UI.md` | Histórico — describe el `SceneManager` previo al vertical slice. Auto-declarado obsoleto en su primera línea. |
| `docs/RELEVAMIENTO.md` | Snapshot previo al vertical slice, superado por `docs/architecture.md`. |
| `docs/prompt.md`, `docs/prompt2.md` | Prompts de trabajo (scratch) para integración de assets, no documentación de arquitectura viva. |

## 3. Arquitectura

Fuente autoritativa: `docs/architecture.md` + ADRs. Estado real confirmado en `CMakeLists.txt` (qué se compila) y headers en `include/cybercba/`.

| Componente | Estado | Ubicación | Rol |
|---|---|---|---|
| `GameApp` | Implementado | `src/app/GameApp.{hpp,cpp}` | Orquestador real: render raylib, navegación entre pantallas (`Screen` enum: Splash/Menu/CharacterSelection/Intro/World/…), posee `InputRouter`, `AudioService`, `DevOverlay`, `SaveService`, `GameSession`. |
| `GameSession` | Implementado | `include/cybercba/GameSession.hpp`, `src/domain/GameSession.cpp` | Contenedor de `CampaignProgress`, `NarrativeState`, `PlayerState`, `AccessibilitySettings`, `AudioSettings`; expone `model()`, `progress()`, `startNewGame()`, `recordQueueResult()`. |
| `GameController` | Implementado | `include/cybercba/GameController.hpp`, `src/application/GameController.cpp` | Capa de aplicación: recibe `GameCommand` (enum: NewGame, ContinueGame, QueuePredict*, QueueExecute, QueueHint, QueueUndo, QueueRestart, QueueAdvanceDebrief) y despacha sobre `GameSession`. |
| `Campaign` (dominio) | Implementado | `include/cybercba/Campaign.hpp`, `src/domain/Campaign.cpp` | Reglas puras: `CharacterDefinition`, `PlayerState`, `NarrativeState`, `CampaignProgress`, `AbilitySystem` (hack/strike/applyDamage), `MissionState`/`PrologueStage`. Testeable sin raylib. |
| `QueueChallenge` | Implementado (educativo, fuera de ficción) | `include/cybercba/QueueChallenge.hpp`, `src/domain/QueueChallenge.cpp` | FIFO de 4 slots sobre `std::array` fijo (no STL dinámico), fases Intuition/Prediction/Guided/Independent/Debrief/Complete, undo por snapshots. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GabrielEValenzuela/CyberCba2077](https://github.com/GabrielEValenzuela/CyberCba2077) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
