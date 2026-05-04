---
trigger: always_on
description: - **NEVER use `console.*`** — blocked by oxlint. Use `logError()` from `@/utils/node/logging` (main process) or `window.electronAPI.logs.logError()` (renderer)
---

# Copilot Instructions for Gerbil

## Hard Rules

- **NEVER use `console.*`** — blocked by oxlint. Use `logError()` from `@/utils/node/logging` (main process) or `window.electronAPI.logs.logError()` (renderer)
- **Always use absolute imports**: `import { X } from '@/components/X'`
- **Never add explicit return types** — rely on TypeScript inference
- **Never create tests, docs, or GitHub workflows**
- **Never add comments** — code should be self-explanatory; no inline comments, no block comments
- **Move helper functions** out of component files into `src/utils/`

## What Gerbil Is

Gerbil is an Electron desktop app that acts as a launcher and GUI for [KoboldCpp](https://github.com/LostRuins/koboldcpp). It is **not** a new LLM backend — it wraps KoboldCpp and makes it usable without touching the terminal.

**The problem it solves**: KoboldCpp is an excellent all-in-one local LLM backend (text gen, image gen, multimodal, agents) but its own launcher UI is bad. Gerbil replaces and significantly improves that launcher.

**Gerbil vs KoboldCpp's launcher**: Gerbil adds auto binary download, GPU auto-detection (CUDA/ROCm/Vulkan/Metal), image gen presets (FLUX, Chroma, Z-Image, Qwen), HuggingFace model search/download, SillyTavern and OpenWebUI auto-launch, config save/load, real-time system monitoring, Cloudflare tunnel support, and a proper desktop experience.

## User Base

- People who want to run LLMs locally with real control over the backend
- SillyTavern users (roleplay/character AI) — Gerbil auto-launches ST alongside KoboldCpp
- Image generation users — Gerbil has first-class image gen with 4 presets
- Power users who want GPU acceleration configured correctly without guesswork

## Stack

Electron + React + Zustand + Mantine + TypeScript + pnpm + oxlint. No test framework.

## Validation

Always run after making changes:

```sh
pnpm check   # oxlint + oxfmt (lint + format check)
```

Fix lint/format issues with `pnpm fix`. No test suite exists.

## Gerbil's Role: KoboldCpp Orchestrator

Gerbil is a manager and orchestrator for KoboldCpp. It doesn't implement LLM inference — it configures, launches, monitors, and wraps KoboldCpp. KoboldCpp releases monthly updates that frequently add new CLI flags and capabilities.

**How KoboldCpp flags surface in Gerbil:**

1. **Promoted to UI** — High-value flags get a proper control (checkbox, slider, file picker) in the Launch screen tabs. These live in `launchConfig` store and are passed as CLI args to KoboldCpp at launch. They must be added to `UI_COVERED_ARGS` in `CommandLineArgumentsModal.tsx` so they're not duplicated in the modal.

2. **Available in the arguments modal** — Everything else is accessible via `CommandLineArgumentsModal` (`src/components/screens/Launch/CommandLineArgumentsModal.tsx`). This modal contains a hardcoded `COMMAND_LINE_ARGUMENTS` array with every KoboldCpp flag, its description, type, category, and aliases. Users can search and add any flag to the "Additional Arguments" field. These are stored as `additionalArguments` in `KoboldConfig`.

**When KoboldCpp adds new flags:** Add entries to `COMMAND_LINE_ARGUMENTS` in `CommandLineArgumentsModal.tsx`. If a flag deserves first-class UI treatment, also wire it into the launch config store, the relevant Launch tab, and add it to `UI_COVERED_ARGS`.

**`UI_COVERED_ARGS`** is a `Set<string>` at the top of `CommandLineArgumentsModal.tsx` — it lists all flags already exposed by the UI so they're filtered out of the modal. Always keep this in sync when promoting a flag to the UI.

## App Structure

**Screen flow**: Welcome → Download → Launch (tabs: General/Performance/Advanced/Image Gen/Network/Config) → Interface (tabs: Terminal/Chat-Text/Chat-Image)

**Supported GPUs**: CUDA, ROCm (via YellowRoseCx fork), Vulkan, Metal (macOS), CPU fallback

**Frontends**: KoboldAI Lite, llama.cpp (embedded in KoboldCpp), SillyTavern (localhost:3000, needs Node.js), OpenWebUI (localhost:8080, needs uv)

**Image gen presets**: FLUX.1-dev, Chroma-unlocked, Z-Image-Turbo, Qwen2.5-VL-7B (image edit)

**CLI mode**: headless binary execution — requires prior GUI setup to configure binary path

## Source Layout

```
src/
├── main/               # Electron main process (Node.js)
│   ├── index.ts        # Entry: routes to CLI or GUI mode
│   ├── gui.ts          # GUI init: window, tray, IPC, lifecycle
│   ├── cli.ts          # Headless mode: spawn binary, pipe stdio
│   ├── ipc.ts          # All ipcMain.handle/on registrations
│   └── modules/        # Feature domains
│       ├── config.ts           # Settings file (~/.config/Gerbil/config.json)
│       ├── hardware.ts         # GPU/CPU detection
│       ├── monitoring.ts       # Real-time CPU/GPU/RAM metrics
│       ├── tray.ts             # System tray icon & menu
│       ├── window.ts           # Main window creation & lifecycle
│       ├── auto-updater.ts     # Electron auto-updater
│       ├── dependencies.ts     # Check npm/uv/npx availability
│       ├── sillytavern.ts      # Auto-launch SillyTavern
│       ├── openwebui.ts        # Auto-launch OpenWebUI
│       └── koboldcpp/          # KoboldCpp-specific

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lone-cloud/gerbil](https://github.com/lone-cloud/gerbil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
