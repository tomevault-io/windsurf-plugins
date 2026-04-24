---
trigger: always_on
description: Voice Key is an Electron + React + TypeScript desktop application for voice-to-text transcription with text injection. This guide provides commands and conventions for agentic coding agents.
---

# AGENTS.md - Voice Key Development Guide

## Overview

Voice Key is an Electron + React + TypeScript desktop application for voice-to-text transcription with text injection. This guide provides commands and conventions for agentic coding agents.

## Project Structure

```
voice-key/
├── electron/                    # Main process (Node.js)
│   ├── main/                   # Core business logic
│   │   ├── README.md          # Main process documentation
│   │   ├── main.ts            # App entry, window mgmt, IPC, PTT orchestration
│   │   ├── hotkey-manager.ts  # Global shortcuts (globalShortcut API)
│   │   ├── iohook-manager.ts  # Low-level keyboard hooks (uiohook-napi)
│   │   ├── asr-provider.ts    # GLM ASR API integration
│   │   ├── text-injector.ts   # Keyboard simulation (nut-js)
│   │   └── config-manager.ts  # Config persistence (electron-store)
│   ├── preload/               # IPC bridge
│   │   ├── README.md
│   │   └── preload.ts         # contextBridge API exposure
│   ├── shared/                # Cross-process code
│   │   ├── README.md
│   │   ├── types.ts           # TypeScript types, IPC channels
│   │   └── constants.ts       # App constants (GLM config, hotkeys)
│   ├── README.md              # Electron overview
│   └── electron-env.d.ts
│
├── src/                        # Renderer process (React)
│   ├── components/            # React components
│   │   ├── ui/               # shadcn/ui component library
│   │   │   ├── README.md
│   │   │   ├── button.tsx    # Multi-variant button
│   │   │   ├── input.tsx     # Text input
│   │   │   ├── card.tsx      # Card container
│   │   │   ├── dialog.tsx    # Modal dialog
│   │   │   ├── select.tsx    # Dropdown select
│   │   │   └── ...           # 13+ more components
│   │   ├── README.md
│   │   └── AudioRecorder.tsx  # Headless audio capture (Web Audio API)
│   ├── pages/                 # Route pages
│   │   ├── README.md
│   │   ├── HomePage.tsx       # Main dashboard (stats, status)
│   │   ├── SettingsPage.tsx   # Config management UI
│   │   └── HistoryPage.tsx    # Transcription history (MVP: empty state)
│   ├── layouts/               # App layouts
│   │   ├── README.md
│   │   └── MainLayout.tsx     # Sidebar nav + content area
│   ├── lib/                   # Utilities
│   │   ├── README.md
│   │   └── utils.ts           # cn() class merger
│   ├── README.md              # Renderer overview
│   ├── App.tsx                # Root component (hash routing)
│   ├── main.tsx               # React entry point
│   ├── index.css              # Global styles (Tailwind + theme vars)
│   ├── global.d.ts            # Window.electronAPI types
│   └── vite-env.d.ts
│
├── public/                     # Static assets
│   └── voice-key-logo.svg
│
├── docs/                       # Architecture & planning docs
│   ├── arch/
│   │   └── architecture-mvp-v3.md
│   └── mvp-plan.md
│
├── package.json               # Dependencies & scripts
├── tsconfig.json              # TypeScript config
├── vite.config.ts             # Vite build config
├── eslint.config.js           # ESLint rules
├── prettier.config.js         # Prettier formatting
├── tailwind.config.ts         # Tailwind CSS config
├── commitlint.config.ts       # Conventional Commits validation
├── README.md                  # Project overview
├── CLAUDE.md                  # This file (AI development guide)
└── LICENSE                    # Elastic License 2.0
```

**Key Directories:**

- `electron/main/` - Core PTT flow: keyboard hooks → recording → ASR → text injection
- `electron/preload/` - Secure IPC bridge between main and renderer processes
- `src/components/ui/` - shadcn/ui library (18 components)
- `src/pages/` - Three main routes: Home, Settings, History

## Documentation Guidelines

### README.md Files

Every directory contains a `README.md` that describes its structure and contents. **These READMEs are critical for understanding the codebase.**

#### When Reading/Searching Code

**ALWAYS read the README.md first** before diving into code:

1. **Start at the target directory** - Open `{directory}/README.md` to understand structure
2. **Read parent READMEs** - If context is unclear, read parent directory READMEs
3. **Use README as a map** - File descriptions in README guide you to relevant code

**Example workflow:**

- Need to understand ASR integration? Read `electron/main/README.md` → Find `asr-provider.ts` description
- Looking for UI components? Read `src/components/ui/README.md` → See component categories
- Exploring IPC? Read `electron/preload/README.md` → Understand exposed APIs

#### When Writing/Modifying Code

**ALWAYS update the README.md** after creating/modifying files:

1. **Update immediately** - Don't defer README updates to "later"
2. **Keep it current** - README must reflect actual current state, not historical plans
3. **Be concise** - Use minimal words to describe purpose clearly
4. **No fluff** - Avoid generic descriptions like "handles X", "manages Y" - be specific

**What to update:**

- **New file?** Add entry with concise description of its role
- **Modified file?** Update description if purpose/behavior changed significantly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BuildWithAIs/voicekey](https://github.com/BuildWithAIs/voicekey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
