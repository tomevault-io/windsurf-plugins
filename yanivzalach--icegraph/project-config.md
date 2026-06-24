---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Change Policy

- Keep all changes as minimal as possible unless explicitly asked for more.
- Never change behavior that was not explicitly requested.
- Apply DRY principles where possible.
- Python files must not exceed 400 lines.
- Never run git commands — only the user does.
- When a change affects the user interface (behavior, navigation, interactions, views), update the relevant section in `frontend/src/pages/DocsPage.jsx`.
- The Issues panel content (errors and warnings) is driven entirely by the backend response (`data.errors`, `data.warnings`). UI changes to this panel must be coordinated with backend error/warning emission logic.
- Whenever creating a constant that is settable via an environment variable, define its default value in `backend/constants.py` and add its description to `README.md`.

## Project Overview

IceGraph is an interactive Apache Iceberg debugging and visualization platform. It provides a hierarchical, graph-based view of Iceberg table metadata to help engineers debug complex table states and trace metadata evolution. It is **read-only** and built exclusively for **Spark Connect** backends targeting **Iceberg Table Version 2**.

## Commands

### Backend (Python + UV)

```bash
cd backend
uv sync                    # Install dependencies
uv run python main.py      # Start Flask server on port 5000
```

### Frontend (Node/React)

```bash
cd frontend
npm i                      # Install dependencies
npm run dev                # Start Vite dev server on port 3000 (proxies /api to port 5000)
npm run build              # Production build to /dist
```

### Docker

```bash
docker build -t icegraph .                                          # Multi-stage build
docker run -e SPARK_REMOTE=sc://<ip>:15002 -p 5000:5000 icegraph   # Run container

cd docker_demo && docker compose up   # Full demo stack with mock Iceberg tables
```

## Architecture

**Backend** (`/backend/`) — Flask API that reads Iceberg metadata via Spark Connect:

- `main.py` — Flask app with API routes; uses `ThreadPoolExecutor` for async job processing
- `collectors/` — Pull Iceberg metadata via Spark: snapshots → metadata files → manifests → data files
- `table_inventory/` — Orchestrates collection into a unified inventory structure
- `search_cutoff/` — Optimizes snapshot iteration range to avoid full scans
- `graph_normalizer/` — Transforms inventory data into graph nodes/links for the frontend
- `extractors/` — Extract useful information from specific file types (manifests, data files) via Spark Connect
- `base_classes/` — Abstractions for files and Spark actions

**Frontend** (`/frontend/src/`) — React SPA:

- Pages: `GraphPage` (force-graph visualization), `MetadataPage`, `TimelinePage`, `FileTreePage`, `SnapshotSelectionPage`
- `TableLayout.jsx` wraps all table-specific pages; `TableSpecsContext.jsx` shares table state
- `graphConstants.js` defines node/link visual constants and `fileTypeLabel()` for human-readable node types
- `uiTypography.js` — shared Tailwind class tokens for labels, body text, inputs, buttons, and toolbar controls
- `components/PanelContent.jsx` — side-panel components (`PanelHeader`, `PanelDetailRow`, `PanelSectionTitle`) and panel-specific typography tokens
- `components/ResizableSidePanel.jsx` — draggable side panel shell used by Graph and Timeline
- `mocks/` — MSW handlers used in the GitHub Pages demo (no real backend)

**API flow:**
1. `GET /api/v1/tables` — list Iceberg tables from the Spark catalog (for Home and navbar picker)
2. `GET /api/v1/snapshot-map/<table>` — load snapshot history for UI selection
3. `POST /api/v1/graph-data` — submit async job with table name + snapshot range
4. `GET /api/v1/graph-data/<job_id>` — poll until complete, returns graph JSON

## Frontend Styling Conventions

Typography and repeated UI patterns live in `frontend/src/uiTypography.js`. Prefer importing tokens from there instead of duplicating Tailwind class strings.

**Token layers:**

| File | Role |
|---|---|
| `uiTypography.js` | App-wide tokens: form labels, muted body text, inputs, primary/toolbar buttons, mono text |
| `PanelContent.jsx` | Side-panel tokens and components; re-exports field-label tokens from `uiTypography.js` |
| `graphConstants.js` | Graph-specific visuals (`NODE_STYLE_MAP`, `fileTypeLabel`) |

**Common tokens:**

- `UI_BODY_MUTED_CLASS` — secondary paragraph text (`text-sm text-slate-400`)
- `UI_FIELD_LABEL_CLASS` — uppercase field labels in panels and metadata rows (caption size, slate-500)
- `UI_FORM_LABEL_CLASS` — uppercase form labels (xs size, slate-400, block)
- `UI_TOOLBAR_BUTTON_BASE` — standard graph toolbar button with `py-2.5`
- `UI_TOOLBAR_BUTTON_LAYOUT` — same toolbar look **without** vertical padding; use for split buttons (e.g. Inspect/Locked) where inner spans supply `py-2.5`
- `toolbarButtonClass(active)` — active/inactive toolbar button variant

**Side panel:** Graph and Timeline both use `ResizableSidePanel` + `PanelContent`. Panel headers call `fileTypeLabel(nodeType)` from `graphConstants.js`. Timeline diff rows use `PANEL_DIFF_*` tokens for Before/After labels and values.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YanivZalach/IceGraph](https://github.com/YanivZalach/IceGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
