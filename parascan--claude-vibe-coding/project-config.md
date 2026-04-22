---
trigger: always_on
description: This file provides guidance for AI assistants (Claude Code and similar) working in this repository. It covers project structure, development workflows, conventions, and best practices.
---

# CLAUDE.md — Claude Vibe Coding

This file provides guidance for AI assistants (Claude Code and similar) working in this repository. It covers project structure, development workflows, conventions, and best practices.

---

## Repository Overview

**Claude-Vibe-Coding** is a repository dedicated to AI-assisted development workflows using Anthropic's Claude models and Claude Code CLI. It serves as a reference implementation and playground for "vibe coding" — a development style where a developer works collaboratively with an AI assistant to rapidly prototype, build, and iterate on software.

The primary application in this repository is a **Workout Generator** — a Progressive Web App (PWA) built with React and TypeScript that generates personalized strength and cardio workout sessions.

---

## Project Structure

```
Claude-Vibe-Coding/
├── CLAUDE.md                          # This file — AI assistant guidance
├── workout.html                       # Standalone single-file HTML build (212 KB)
├── .github/
│   └── workflows/
│       └── deploy.yml                 # GitHub Pages CI/CD pipeline
└── workout-app/                       # Main React + TypeScript application
    ├── index.html                     # HTML entry point
    ├── package.json                   # Dependencies & scripts
    ├── vite.config.ts                 # Vite config with PWA plugin
    ├── tsconfig.json                  # TypeScript base config
    ├── tsconfig.app.json              # App-specific TS config
    ├── tsconfig.node.json             # Node/build TS config
    ├── eslint.config.js               # ESLint configuration
    ├── public/
    │   ├── apple-touch-icon.png       # iOS app icon
    │   └── icons/
    │       ├── icon-192.png           # PWA icon (192px)
    │       └── icon-512.png           # PWA icon (512px)
    └── src/
        ├── App.tsx                    # Root component with top-level state
        ├── main.tsx                   # React entry point
        ├── App.css                    # Main styles
        ├── index.css                  # Global styles
        ├── components/
        │   ├── ModeSelect.tsx         # Strength vs Cardio mode picker
        │   ├── TimeInput.tsx          # Duration input component
        │   ├── WorkoutDisplay.tsx     # Strength workout plan display
        │   ├── WorkoutTimer.tsx       # Live strength timer with audio cues
        │   ├── ExerciseCard.tsx       # Individual exercise card with demo images
        │   ├── CardioDisplay.tsx      # Cardio session selection UI
        │   ├── CardioRoutePlanner.tsx # Map-based route toggle (loop / out-and-back)
        │   └── CardioTimer.tsx        # Live cardio interval timer with audio
        ├── data/
        │   ├── exercises.ts           # 40+ exercise definitions with demo images
        │   └── cardio.ts              # Cardio activity definitions (road & Peloton)
        └── lib/
            ├── generator.ts           # Strength workout generation algorithm
            └── cardioGenerator.ts     # Cardio session generation algorithm
```

---

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| UI framework | React | 19.2.0 |
| Language | TypeScript | 5.9.3 |
| Build tool | Vite | 7.3.1 |
| Maps | Leaflet + react-leaflet | 1.9.4 / 5.0.0 |
| PWA | vite-plugin-pwa | 1.2.0 |
| Linting | ESLint | 9.39.1 |
| Deployment | GitHub Pages | — |

---

## Application Features

### Strength Mode

- Generates time-based dumbbell circuit workouts (10–60+ minutes)
- Multi-round structure with configurable work/rest/transition periods
- 40+ exercises targeting all major muscle groups
- Equipment: SelectTech 552 dumbbells with weight recommendations
- Demo images for each exercise (fetched from free exercise database)
- Estimated calorie burn (~8 cal/min)
- Live workout timer with slide animations and spoken audio cues

### Cardio Mode

- Road running and Peloton cycling session plans
- Interval generation based on total session duration
- Effort-based descriptions: easy / moderate / hard
- Workout types: HIIT, tempo, fartlek, steady state
- Live interval timer with spoken audio cues
- Route planner: map-based loop vs. out-and-back toggle with distance in miles

### PWA / Offline

- Installable on iOS and Android home screens
- Offline support via service worker caching
- Standalone single-file HTML build (`workout.html`) as a fallback

---

## Development Workflows

### Getting Started

1. Clone the repository and navigate to the app:
   ```bash
   git clone <repo-url>
   cd Claude-Vibe-Coding/workout-app
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```
   The app is served at `http://localhost:5173` with Hot Module Replacement.

### Available Scripts

Run all commands from inside `workout-app/`:

```bash
npm run dev       # Start Vite dev server (HMR enabled)
npm run build     # TypeScript type-check + Vite production build
npm run lint      # ESLint check across src/
npm run preview   # Preview production build locally
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/parascan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
