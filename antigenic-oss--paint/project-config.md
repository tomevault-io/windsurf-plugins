---
trigger: always_on
description: Visual design editor for localhost web projects. Inspect elements,
---

# pAInt — Development Guidelines

Visual design editor for localhost web projects. Inspect elements,
edit styles, manage CSS variables, and generate changelogs for
Claude Code — all from a visual-editor-style three-column dark UI.

## How to Use

### Quick Start

1. Start pAInt: `bun dev` (runs on `http://localhost:4000` by default)
2. Start your target project's dev server (e.g., `http://localhost:3000`)
3. Open pAInt in your browser
4. Select your target's localhost port from the dropdown in the top bar and click **Connect**
5. The target page loads in the center iframe — start inspecting and editing

### Connecting to Your Project

There are **three ways** to connect pAInt to your project:

#### Method 1: Automatic (Service Worker Proxy) — Recommended
When you click **Connect**, pAInt loads your target page through a Service Worker proxy. The proxy automatically intercepts requests, injects the inspector script, and strips security headers — all in the browser. No script tags or project modifications needed. Your page's scripts and client-side rendering work normally.

#### Method 2: Vercel Deployment (Bridge Mode)
When pAInt is deployed to Vercel, run the local bridge server (`bun run bridge`) on port 4002. The bridge handles proxy requests, project scanning, and Claude CLI execution on the user's machine.

### Typical Workflow

```
Open pAInt → Connect to localhost project
       ↓
Inspect elements (hover/click in preview or click in Layers tree)
       ↓
Edit styles in the right panel (typography, spacing, colors, layout)
       ↓
Test across breakpoints (Mobile / Tablet / Desktop)
       ↓
Review tracked changes in the Changes tab
       ↓
Export changelog → Paste into Claude Code → Changes applied to source files
```

## Tech Stack

- **Runtime / Package Manager**: Bun (`bun dev`, `bun run build`, `bun install`)
- **Framework**: Next.js 15 App Router (TypeScript)
- **Styling**: Tailwind CSS 4 — `class` dark mode strategy, CSS custom properties
- **State**: Zustand 5 with slices (`elementSlice`, `changeSlice`, `uiSlice`, `treeSlice`, `claudeSlice`, `cssVariableSlice`, `componentSlice`, `consoleSlice`, `terminalSlice`)
- **Terminal**: xterm.js + node-pty
- **Communication**: `window.postMessage` between editor (parent) and inspector (iframe)
- **Persistence**: `localStorage` (changes keyed by target URL, recent URLs, settings)

## Project Structure

```
src/
├── app/
│   ├── layout.tsx                     # Root layout, dark mode, providers
│   ├── page.tsx                       # Main editor (three-column layout)
│   ├── globals.css                    # Tailwind entry + dark mode variables
│   ├── docs/                          # Setup guide page (/docs)
│   │   ├── page.tsx
│   │   ├── DocsClient.tsx
│   │   └── layout.tsx
│   └── api/
│       ├── proxy/[[...path]]/route.ts # Reverse proxy to target localhost
│       ├── claude/
│       │   ├── analyze/route.ts       # Claude CLI read-only analysis
│       │   ├── apply/route.ts         # Claude CLI write mode
│       │   ├── scan/route.ts          # AI-powered project scan
│       │   ├── pick-folder/route.ts   # Folder picker
│       │   └── status/route.ts        # CLI availability check
│       ├── project-scan/
│       │   ├── route.ts               # Project structure scanning
│       │   ├── css-variables/route.ts # CSS variable extraction
│       │   └── tailwind-config/route.ts # Tailwind config detection
│       └── project/scan/route.ts      # Project directory scanning
├── components/
│   ├── Editor.tsx                     # Three-column shell
│   ├── TopBar.tsx                     # URL input, breakpoints, actions
│   ├── TargetSelector.tsx             # Localhost URL bar + connect + status dot
│   ├── BreakpointTabs.tsx             # Mobile | Tablet | Desktop
│   ├── ResponsiveToolbar.tsx          # Responsive controls toolbar
│   ├── PageSelector.tsx               # Page navigation dropdown
│   ├── PreviewFrame.tsx               # Iframe container
│   ├── ConnectModal.tsx               # Connection setup modal
│   ├── ChangeSummaryModal.tsx         # Change summary overlay
│   ├── ProjectFolderBanner.tsx        # Project folder selection banner
│   ├── left-panel/
│   │   ├── LeftPanel.tsx              # Left panel container
│   │   ├── IconSidebar.tsx            # Icon sidebar (Layers/Pages/Components/Add/Terminal)
│   │   ├── LayersPanel.tsx            # DOM tree navigator
│   │   ├── LayerNode.tsx              # Individual tree node
│   │   ├── LayerSearch.tsx            # Tree search
│   │   ├── PagesPanel.tsx             # Page navigation
│   │   ├── ComponentsPanel.tsx        # React component browser
│   │   ├── AddElementPanel.tsx        # HTML element inserter
│   │   ├── icons.tsx                  # Panel icons
│   │   └── terminal/
│   │       ├── TerminalPanel.tsx      # Embedded xterm.js terminal
│   │       └── ScanOverlay.tsx        # Scan progress overlay
│   ├── right-panel/
│   │   ├── RightPanel.tsx             # Right panel container
│   │   ├── PanelTabs.tsx              # Tab switcher (Design/Variables/Changes/Claude/Console)
│   │   ├── ElementLogBox.tsx          # Element info display
│   │   ├── design/                    # Style editing sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Antigenic-OSS/pAInt](https://github.com/Antigenic-OSS/pAInt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
