---
trigger: always_on
description: SpecForge is a web-based drag-and-drop SAP screen mockup tool. It lets SAP consultants
---

# SpecForge — SAP Mockup Studio
## Master Instructions for Claude Code Agent

---

## What this project is

SpecForge is a web-based drag-and-drop SAP screen mockup tool. It lets SAP consultants
prototype any SAP screen (Classic GUI, Fiori 3, or S/4HANA Horizon) in minutes —
without Figma, without SAP access, without a developer.

It fills the gap left by SAP Build (sunset 2020), which had 1,000+ prototypes/week
at peak. No replacement exists for custom (non-Fiori Elements) SAP screen prototyping.

---

## Tech stack — do not deviate from this

| Layer        | Technology                        | Reason                                      |
|--------------|-----------------------------------|---------------------------------------------|
| Frontend     | React 18 + TypeScript             | Component model suits drag-drop canvas      |
| Build        | Vite                              | Fast HMR, simple config                     |
| UI (app UI)  | Tailwind CSS                      | Our own tool's UI                           |
| SAP Fiori    | UI5 Web Components v2.20.0        | Real SAP components, Apache 2.0 licensed    |
| State        | Zustand                           | Simple, no boilerplate                      |
| Drag & drop  | dnd-kit                           | Best React DnD, accessible                  |
| Routing      | React Router v6                   | SPA routing                                 |
| Backend      | FastAPI (Python)                  | AI endpoints, project save/load             |
| Database     | SQLite (dev) / PostgreSQL (prod)  | Project persistence                         |
| AI           | Anthropic Claude API (claude-sonnet-4-5) | Interview engine, slot filling      |
| Export       | html2canvas + jsPDF               | PNG/PDF export of mockups                   |

---

## Project structure

```
specforge/
├── CLAUDE.md                  ← this file (read first, always)
├── AGENTS.md                  ← agent roles and task ownership
├── PLAN.md                    ← phased build plan with tasks
├── docs/
│   ├── SAP_UI_VERSIONS.md     ← SAP GUI vs Fiori 3 vs Horizon specs
│   ├── COMPONENTS.md          ← every SAP component, its slots, props
│   ├── FLOORPLANS.md          ← SAP floorplan layouts and rules
│   ├── SLOT_SCHEMA.md         ← JSON schema for AI slot filling
│   └── API.md                 ← backend API contract
├── src/
│   ├── components/
│   │   ├── canvas/            ← drag-drop canvas and grid
│   │   ├── palette/           ← component sidebar palette
│   │   ├── properties/        ← right-panel property editor
│   │   ├── sap-classic/       ← Classic SAP GUI component renderers
│   │   ├── sap-fiori3/        ← Fiori 3 / Quartz component renderers
│   │   ├── sap-horizon/       ← Horizon / S/4HANA component renderers
│   │   └── shared/            ← shared app UI components
│   ├── stores/
│   │   ├── canvasStore.ts     ← canvas state (components, positions)
│   │   ├── projectStore.ts    ← project metadata, save/load
│   │   └── uiStore.ts         ← app UI state (active panel, mode)
│   ├── agents/
│   │   ├── interviewAgent.ts  ← AI interview conversation manager
│   │   ├── slotFillerAgent.ts ← maps FS JSON → component slot values
│   │   └── specAgent.ts       ← generates FS/TS documents from interview
│   ├── lib/
│   │   ├── sapTables.ts       ← SAP table/field knowledge base
│   │   ├── floorplans.ts      ← floorplan template definitions
│   │   ├── componentRegistry.ts ← all draggable components + metadata
│   │   └── export.ts          ← PNG/PDF/shareable link export
│   └── pages/
│       ├── Home.tsx            ← project list
│       ├── Builder.tsx         ← main canvas page
│       ├── Interview.tsx       ← AI interview flow
│       └── Preview.tsx         ← fullscreen shareable preview
└── backend/
    ├── main.py                ← FastAPI app entry
    ├── routers/
    │   ├── projects.py        ← CRUD for projects
    │   ├── interview.py       ← Claude API interview endpoint
    │   └── export.py          ← server-side export
    ├── models.py              ← SQLAlchemy models
    └── claude_client.py       ← Anthropic SDK wrapper
```

---

## Core concepts — understand these before writing any code

### 1. UI Version
Every element on the canvas belongs to one of three SAP UI versions:
- `classic` — SAP GUI (ECC, old Windows look, #003399 blue, monospace fonts)
- `fiori3` — Fiori 3 / Quartz Light (flat, white, SAP blue #0070F2)
- `horizon` — Morning Horizon / S/4HANA current (rounded, #0064D9, softer)

A project has ONE ui_version. All components on canvas must match it.
Never mix ui_versions on one canvas.

### 2. Floorplan
A floorplan is a layout shell — the structural skeleton of the screen.
Components are placed INSIDE floorplan regions (header, content, footer, sidebar).
Available floorplans per version:

**Classic:** SelectionScreen, ALVGrid, DialogTransaction, DocumentDisplay
**Fiori 3:** ListReport, ObjectPage, OverviewPage, Worklist, AnalyticalList
**Horizon:** Same as Fiori 3 + Flexible Column Layout

### 3. Component
A component is a draggable SAP UI element. Each has:
- `type` — e.g. "input", "checkbox", "table", "button"
- `version` — which ui_version it belongs to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SAP-SHARKS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
