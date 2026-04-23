---
trigger: always_on
description: > **PURPOSE:** This file gives Claude (or any AI coding assistant) full context about this project.
---

# CLAUDE.md — Syndnet Corp / ScoutGPT

> **PURPOSE:** This file gives Claude (or any AI coding assistant) full context about this project.
> Paste this at the start of every Claude.ai chat session, or place it in your repo root for Claude Code to read automatically.
>
> **LAST UPDATED:** [UPDATE THIS DATE EVERY TIME YOU EDIT]
> **CURRENT PHASE:** [UPDATE THIS EVERY SESSION — e.g., "Phase 5B - Universal Parcel Clicking"]

---

## What This Project Is

ScoutGPT is an AI-powered commercial real estate intelligence platform built by Syndnet Corp. Users type natural language queries ("show me 2-4 acre parcels in Travis County") and the platform returns property data, renders map pins, and generates analysis — all through a chat interface with an integrated Mapbox map.

---

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Frontend Framework | React (JavaScript, no TypeScript) | 18.2 |
| Build Tool | Vite | 5.0 |
| Styling | Tailwind CSS | 3.4 |
| Animation | Framer Motion | 10.16 |
| Routing | React Router DOM | 6.0 |
| State Management | React Context API + localStorage | — |
| HTTP Client | Axios | 1.8 |
| Maps | Mapbox GL JS + Mapbox GL Draw | 3.13 |
| Charts | Recharts + D3 | 2.15 / 7.9 |
| Forms | React Hook Form | 7.55 |
| Icons | Lucide React | — |
| Backend Framework | FastAPI (Python 3.11+) + Uvicorn | 0.104 |
| ORM | SQLAlchemy 2.0 (async) + GeoAlchemy2 | — |
| Database | PostgreSQL 15 + PostGIS 3.3 (Neon) | — |
| Schema Management | Prisma (100+ models, ~1689 lines) | — |
| AI | Claude API (Anthropic) | — |
| MCP Servers | FastMCP (Python) | — |
| Frontend Hosting | Netlify (auto-deploy from main) | — |
| Backend Hosting | Render | — |
| MCP Hosting | Render | — |

---

## Repository Locations (Local Mac)

```
~/scoutgpt_9461/              ← Frontend (React/Vite)
~/scoutgptpro-backend/        ← Backend (FastAPI/Express hybrid)
~/scoutgpt-mcps/              ← MCP servers
  ├── property-mcp-python/    ← Property data MCP (deployed)
  └── gis-mcp-python/         ← GIS data MCP (not started)
```

---

## Deployment URLs

| Service | URL | Platform |
|---------|-----|----------|
| Frontend | https://scoutcrm.netlify.app | Netlify |
| Backend | https://scoutgpt-backend.onrender.com | Render |
| Property MCP | https://scoutgpt-property-mcp.onrender.com/mcp/ | Render |

---

## Frontend Structure (`~/scoutgpt_9461`)

```
src/
├── pages/
│   └── scout-ai-chat/
│       ├── index.jsx                    # Main chat page (entry point)
│       └── components/
│           └── MapWorkspace.jsx         # Mapbox map container
├── components/
│   ├── chat/
│   │   ├── ChatPanel.jsx               # Main chat interface
│   │   ├── PropertyCard.jsx            # Individual property card
│   │   └── PropertyCardsList.jsx       # List of property results
│   ├── layout/
│   │   ├── tabs/
│   │   │   └── ScoutTab.jsx            # Chat tab component
│   │   └── ConsolidatedPanel.jsx       # Left panel container
│   ├── property/
│   │   └── PropertyInfoCard.jsx        # Detailed property info
│   └── artifacts/
│       └── ...                         # Artifact renderers
├── hooks/
│   ├── useChatApi.js                   # Chat API integration
│   ├── useChatHistory.js               # Chat session management
│   └── useMapLayerRenderer.js          # GIS layer rendering
├── utils/
│   └── queryClassifier.js              # Intent classification (GIS vs property)
├── services/
│   └── ...                             # API service layer
└── styles/
    └── popup.css                       # Map popup styles
```

---

## Backend Structure (`~/scoutgptpro-backend`)

```
src/
├── routes/
│   ├── chat.js              # Main Claude chat endpoint
│   ├── ai.js                # AI query pipeline (Claude + MCP)
│   ├── parcels.js           # Parcel CRUD operations
│   ├── gis.js               # GIS layer queries
│   ├── boundaries.js        # ZIP boundary choropleth data
│   ├── artifacts.js         # Artifact generation
│   └── ... (30 route files total)
├── tools/
│   ├── index.js             # Claude tool definitions (7 tools)
│   └── handlers.js          # Tool execution handlers
├── services/
│   ├── mcp/
│   │   ├── server-manager.js    # MCP connection manager
│   │   └── tool-router.js       # Routes tools to correct MCP server
│   ├── claude-writeback/        # Session/message persistence
│   ├── artifacts/               # PDF/CSV/XLSX generators
│   ├── enrichment/              # Property analysis orchestrator
│   └── webSearch/               # Brave API integration
├── db/                      # Database pool config
└── server.js                # Express app entry point

prisma/
├── schema.prisma            # 100+ models, ~1689 lines
└── migrations/              # SQL migration files
```

---

## MCP Server (`~/scoutgpt-mcps/property-mcp-python`)

**4 working tools:**
- `get_property(parcel_id)` — Single property lookup
- `search_properties(county, filters)` — Multi-county filtered search
- `get_enrichment(parcel_id)` — Enrichment data for a parcel
- `bulk_properties(parcel_ids)` — Batch lookup (max 1000)

**Endpoint:** `https://scoutgpt-property-mcp.onrender.com/mcp/` (trailing slash required)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Syndnet-CRE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
