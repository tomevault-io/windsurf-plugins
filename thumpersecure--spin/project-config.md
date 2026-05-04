---
trigger: always_on
description: This document provides essential context for AI assistants working on this codebase.
---

# CLAUDE.md - AI Assistant Guide for Spin

This document provides essential context for AI assistants working on this codebase.

## Project Overview

**Spin** is a privacy-first OSINT (Open Source Intelligence) investigation browser. Each version uses a different detective/investigator theme. The current release is **v12.0.3 "Jessica Jones"**.

### Version History
| Version | Codename | Stack | Status |
|---------|----------|-------|--------|
| v12.0 | Jessica Jones | Tauri 2.0 + React 19 + Rust | **Active Development** |
| v5.0 | The Multiple Man | Tauri 2.0 + React 19 + Rust | Superseded by v12 |
| v4.x | The Exorcist's Edge | Electron + Svelte 5 | Legacy (in `src/`) |

### Core Concepts
- **Identity Dupes**: Multiple isolated browser identities with unique fingerprints
- **Hivemind**: Real-time entity synchronization across identities
- **MCP Agents**: AI-powered investigation assistants (Model Context Protocol)
- **Dynamic Privacy Engine**: Automatic OPSEC level adjustment based on site risk
- **CEF Integration**: Embedded Chromium with per-identity fingerprint control
- **Session Cloning**: Secure session transfer between identities with SHA-256 verification
- **Investigation Timeline & Graph**: D3.js entity relationship visualization

## Repository Structure

```
Spin/
├── app/                          # v12.0 Tauri+React (ACTIVE DEVELOPMENT)
│   ├── src/                      # React TypeScript frontend
│   │   ├── components/           # UI components
│   │   │   ├── browser/          # TitleBar, TabBar, NavBar, BrowserView
│   │   │   ├── identity/         # IdentityPanel
│   │   │   ├── hivemind/         # HivemindPanel
│   │   │   ├── investigation/    # InvestigationPanel, Timeline, Graph
│   │   │   ├── mcp/              # McpPanel
│   │   │   ├── osint/            # OsintPanel
│   │   │   ├── privacy/          # PrivacyDashboard
│   │   │   └── ui/               # SidePanel, SettingsPanel
│   │   ├── store/                # Redux Toolkit
│   │   │   ├── index.ts          # Store configuration
│   │   │   └── slices/           # Feature slices (9 slices)
│   │   └── theme/                # Mantine theme config
│   ├── src-tauri/                # Rust backend
│   │   ├── src/
│   │   │   ├── commands/         # Tauri IPC command handlers
│   │   │   ├── core/             # Business logic (identity, fingerprint, privacy)
│   │   │   ├── cef/              # Chromium Embedded Framework
│   │   │   ├── session/          # Session cloning
│   │   │   ├── investigation/    # Timeline & graph
│   │   │   ├── hivemind/         # Entity sync system
│   │   │   ├── mcp/              # MCP server / Claude API integration
│   │   │   └── storage/          # sled database wrapper
│   │   └── tauri.conf.json       # Tauri configuration
│   └── package.json              # App dependencies
│
├── src/                          # v4.x Electron+Svelte (LEGACY)
├── tests/                        # Jest test files
├── scripts/                      # Build scripts
├── assets/                       # Application icons
├── .github/workflows/            # CI/CD configuration
└── package.json                  # Root dependencies
```

## Tech Stack

### v12 (app/) - Active Development
| Layer | Technology | Version |
|-------|------------|---------|
| Runtime | Tauri | 2.0 |
| Frontend | React | 19.x |
| Language | TypeScript | 5.x |
| State | Redux Toolkit | 2.x |
| UI Library | Mantine | 8.x |
| Icons | Tabler Icons | 3.x |
| Backend | Rust | 1.75+ |
| Database | sled | Embedded KV |
| AI | Claude API (MCP) | Messages API |
| Build | Vite | 7.x |

## Development Commands

```bash
cd app

# Development with hot reload
npm run tauri:dev

# Production build
npm run tauri:build

# Frontend only (React)
npm run dev
npm run build

# Linting
npm run lint
```

## Code Conventions

### TypeScript (Frontend)

**Components**
```typescript
// PascalCase for components, placed in feature directories
// src/components/feature/ComponentName.tsx

import { useAppSelector, useAppDispatch } from '../../store';

function ComponentName() {
  const dispatch = useAppDispatch();
  const data = useAppSelector((state) => state.feature.data);
  // ...
}

export default ComponentName;
```

**Redux Slices**
```typescript
// src/store/slices/featureSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface FeatureState {
  items: Item[];
  loading: boolean;
}

const featureSlice = createSlice({
  name: 'feature',
  initialState,
  reducers: {
    actionName: (state, action: PayloadAction<PayloadType>) => {
      // Immer mutations allowed
    },
  },
});
```

### Rust (Backend)

**Tauri Commands**
```rust
// src-tauri/src/commands/feature.rs
use tauri::command;

#[command]
pub async fn command_name(param: String) -> Result<ReturnType, String> {
    // Implementation
    Ok(result)
}
```

**Module Organization**
```rust
// src-tauri/src/feature/mod.rs
pub mod submodule;

pub fn init(app_handle: &tauri::AppHandle) -> Result<(), Box<dyn std::error::Error>> {
    // Initialization logic
    Ok(())
}
```

## Architecture Patterns

### State Management
- **Redux Toolkit** with 9 feature slices
- Typed hooks: `useAppDispatch`, `useAppSelector`
- Async actions via `createAsyncThunk`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thumpersecure/Spin](https://github.com/thumpersecure/Spin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
