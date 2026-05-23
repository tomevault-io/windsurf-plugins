---
trigger: always_on
description: Kompetitives Kartenspiel: Zwei Spieler (Mensch vs KI) kompilieren um die Wette ihre 3 Protokolle. Datengetriebenes System mit 32+ Protokollen als JSON-Definitionen.
---

# COMPILE Card Game

Kompetitives Kartenspiel: Zwei Spieler (Mensch vs KI) kompilieren um die Wette ihre 3 Protokolle. Datengetriebenes System mit 32+ Protokollen als JSON-Definitionen.

## Tech Stack
- **Framework**: React 19 + Vite 6
- **Sprache**: TypeScript 5.8 (strict mode)
- **Styling**: Vanilla CSS mit CSS-Variablen (Cyberpunk-Theme)
- **Testing**: Vitest (Unit), Playwright (E2E)
- **Datenbank**: Keine (Client-only, localStorage fuer Statistiken)

## Projektstruktur
```
compile/
├── components/           # React UI-Komponenten (Card, Lane, GameBoard, Modals, AnimationOverlay)
├── screens/              # Hauptbildschirme (GameScreen, MainMenu, ProtocolSelection)
│   └── CustomProtocolCreator/  # Protocol-Editor mit Effect-Editoren
├── hooks/                # React Hooks (useGameState, useStatistics)
├── contexts/             # React Contexts (AnimationQueueContext)
├── logic/                # Gesamte Spiellogik (KEINE async Operationen!)
│   ├── ai/               # KI-Systeme (easy, normal, hardImproved)
│   ├── animation/        # animationHelpers.ts (Factories) + aiAnimationCreators.ts (DRY Helper)
│   ├── customProtocols/  # effectInterpreter.ts (Haupt-Entry), Protocol-Loader
│   ├── effects/actions/  # Modulare Executoren (drawExecutor, flipExecutor, deleteExecutor, etc.)
│   ├── game/             # phaseManager, aiManager, reactiveEffectProcessor, stateManager
│   │   ├── resolvers/    # cardResolver, playResolver, discardResolver, handCardResolver, etc.
│   │   └── helpers/      # actionUtils (findCardOnBoard, isCardUncovered, etc.)
│   ├── keywords/         # Keyword-Handler
│   └── utils/            # log, boardModifiers, logMessages
├── custom_protocols/     # JSON-Definitionen fuer alle Protokolle (32+)
├── types/                # index.ts (GameState, PlayedCard, etc.) + animation.ts (AnimationQueueItem, etc.)
├── utils/                # snapshotUtils, targeting
├── constants/            # animationTiming (Durations, Stagger, Easings)
├── styles/               # CSS-Dateien
├── tests/                # Vitest Unit-Tests
└── e2e/                  # Playwright E2E-Tests
```

## Entwicklung
```bash
npm run dev              # Entwicklungsserver (NUR DER USER STARTET DIES! NIEMALS CLAUDE!)
npm run build            # Production Build (zum Pruefen von Kompilierungsfehlern)
npm test                 # Unit-Tests (Vitest)
npm run test:watch       # Tests im Watch-Modus
npm run test:e2e         # Playwright E2E-Tests (startet dev server automatisch)
npm run check:effects    # Prueft pending Effects
npm run test:protocols   # Testet Custom Protocols
```

## Spielregeln (Kurzfassung)

### Gewinnbedingung
Alle 3 eigenen Protokolle kompilieren.

### Turn-Flow (6 Phasen)
```
1. Start Phase    -> "start:" Effekte auf face-up Karten mit sichtbarem Effekt ausfuehren
2. Control Phase  -> Fuehrt in 2+ Lanes? -> Erhaelt Control Component
3. Compile Phase  -> Lane >= 10 UND > Gegner -> MUSS kompilieren
4. Action Phase   -> PLAY (Karte spielen) ODER REFRESH (Hand auf 5 auffuellen)
5. Hand Limit     -> Auf 5 Karten abwerfen wenn noetig
6. End Phase      -> "end:" Effekte auf face-up Karten mit sichtbarem Effekt ausfuehren
```
**Sichtbarkeit**: Top-Effekte sind sichtbar wenn face-up (auch covered). Bottom-Effekte nur wenn face-up UND uncovered.

### Kompilierung
- **Bedingungen**: Lane-Wert >= 10 UND hoeher als Gegner in gleicher Lane
- **Ablauf**: Alle Karten beider Seiten in Lane geloescht -> Protokoll wird "Compiled"
- **Recompile**: Bereits kompiliert? -> Karten geloescht, STATT Umdrehen: Ziehe 1 vom Gegnerdeck (ownership change!)
- **Control Component**: Wer in 2+ Lanes fuehrt, erhaelt Control. Bei Compile/Refresh mit Control: Darf Protokolle rearrange (Positions/Status tauschen)

### Karten-Werte
- **Face-up**: Angezeigter Wert (0-6+)
- **Face-down**: Immer Wert 2
- **Face-up spielen**: NUR in matching Protocol Lane, Middle Command triggert
- **Face-down spielen**: In jede Lane, kein Effekt-Trigger

### Targeting-Regeln (KRITISCH)
- **Default**: NUR uncovered (oberste Karte im Stack = `lane[lane.length - 1]`)
- **Covered**: Alle Karten darunter - NICHT waehlbar, es sei denn explizit ("flip 1 covered card")
- **Scope**: "1 card" = eigene ODER gegner | "your card" = nur eigene | "opponent card" = nur gegner

### Karten-Text-Types
1. **Top Command** (Persistent): Aktiv solange face-up (auch wenn covered - Text ist oben immer sichtbar)
2. **Middle Command** (Immediate): Triggert bei Play face-up / Flip face-up / Uncover
3. **Bottom Command** (Auxiliary): Triggered effects, nur wenn face-up UND uncovered

### Interrupt-System
"Last in, first out" - Neueste Effekte werden zuerst abgehandelt. Wenn ein Effekt einen weiteren triggert, wird der neue Effekt zuerst komplett resolved, dann der urspruengliche fortgesetzt.

## Architektur-Ueberblick

### Datenfluss
```
User/AI Aktion -> Resolver (synchron) -> neuer GameState
                                      -> AnimationRequests
                                      -> actionRequired (wartet auf Input)

AnimationRequests -> _pendingAnimationRequests (auf State)
                  -> useEffect erkennt -> enqueueAnimationsFromRequests()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheApo/compile](https://github.com/TheApo/compile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
