---
trigger: always_on
description: - **Name**: HASTE (High-speed Assessment and Satellite Tracking for Emergencies)
---

# HASTE — Repository-Wide Copilot Instructions
#
# These instructions are automatically loaded into EVERY Copilot session
# (VS Code, Copilot CLI, Cloud Agent, Code Review).
# Keep them concise and universally applicable.

## Project Overview

- **Name**: HASTE (High-speed Assessment and Satellite Tracking for Emergencies)
- **Description**: AI-driven framework for rapid disaster assessment using satellite and remote sensing data
- **Primary Languages**: Python 3.11 (backend/core), JavaScript/React (UI)
- **Package Managers**: pip/conda (Python), npm (UI)
- **Owner**: microsoft/haste

## Architecture

```
React UI (Vite + FluentUI + Azure Maps + MSAL)
  └─ Azure Static Web Apps / SWA CLI
       ├─ hastefuncapi (41 HTTP routes, Azure Functions, Python)
       ├─ titilerfuncapi (TiTiler/FastAPI, COG tile serving)
       └─ hastefuncqueues (7 queue triggers, Azure Functions, Python)
            └─ hastegeo core library (hastelib/)
                 ├─ Config · Models · Processors · Data Layers
                 ├─ Runners (Azure Batch GPU) · Utils · Workflows
                 └─ Storage: Blob, Cosmos DB, Data Lake, PostgreSQL
```

## Build & Test

- **Core library build**: `cd hastelib && hatch build -t wheel`
- **Core library tests**: `cd hastelib && hatch run test:pytest`
- **Core library tests (specific file)**: `cd hastelib && hatch run test:pytest tests/path/to/test_file.py -v`
- **UI build**: `cd ui && npm run build`
- **UI lint**: `cd ui && npm run lint`
- **API local run**: `cd api/hastefuncapi && func host start`
- **UI local run**: `cd ui && swa start --app-devserver-url http://localhost:5173 --run 'npm run dev'`

Always run tests before marking a task as complete.
Always run lint before committing changes.

## Coding Standards

### Python (backend, core library)
- Python 3.11+. Use type hints everywhere.
- Pydantic models for data validation (already used throughout).
- Follow existing patterns in `hastegeo.core.processors` for business logic.
- Follow existing patterns in `hastegeo.core.data_layer` for storage backends.
- GDAL/rasterio for geospatial operations — never use raw file I/O for imagery.
- Never hardcode Azure connection strings or keys. Use `Config` class from `hastegeo.core.config`.

### JavaScript/React (UI)
- React functional components with hooks.
- FluentUI component library — do not introduce alternative UI frameworks.
- Azure Maps for all geospatial visualization — no Leaflet/Mapbox.
- MSAL for authentication — do not bypass or mock in production paths.

### General
- Write clear, self-documenting code. Avoid unnecessary comments.
- Prefer small, focused functions over long procedural blocks.
- Handle errors explicitly — do not silently swallow exceptions.
- Validate inputs at system boundaries; trust internal data.
- Never commit secrets, credentials, or API keys.

## Key Domain Concepts

- **Project**: A disaster assessment campaign (e.g., "Maui Wildfires 2023")
- **Image Layer**: A set of pre/post-event satellite imagery for a project
- **Source Type**: Satellite imagery provider (Planet, Maxar, Airbus, etc.)
- **Label Project**: Human labeling of damage on imagery tiles
- **Model**: ML model trained on labeled data for damage classification
- **Inference**: Running a trained model on new imagery
- **Artifact**: Model weights, predictions, and outputs
- **COG**: Cloud Optimized GeoTIFF — the standard imagery format

## Git Conventions

- Default branch: `main`
- Use conventional commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`
- Keep PRs focused — one logical change per PR.
- Write descriptive PR titles and descriptions.

## Specification System

All feature work, refactors, and architecture decisions are driven by specs in `spec/`.

### Structure

```
spec/
├── architecture/
│   ├── overview.md              # System architecture (canonical reference)
│   └── decisions/               # ADRs: 0001-template.md, 0002-xyz.md
├── features/                    # One subfolder per feature spec
│   └── <feature-name>/
│       ├── README.md            # Overview, status, components affected
│       ├── plan.md              # Execution plan, phases, milestones
│       ├── impact-analysis.md   # Risk, dependencies, blast radius
│       ├── user-stories.md      # User stories & acceptance criteria
│       ├── design.md            # Technical design & API contracts
│       ├── data-model.md        # Cosmos DB / Blob / Data Lake schema changes
│       ├── test-plan.md         # Test strategy & coverage matrix
│       └── rollout.md           # Rollout strategy, flags, rollback
└── _templates/                  # Copy templates when starting new work
    ├── feature/                 # Full feature spec template
    └── modification/            # Lighter template for refactors/migrations
```

### Spec Workflow

1. **Before implementing**: Check `spec/features/` for the relevant spec. Read the `design.md` and `user-stories.md` first.
2. **During implementation**: Validate work against the spec's acceptance criteria. Update `plan.md` status as tasks complete.
3. **Architecture decisions**: Record in `spec/architecture/decisions/` using the ADR template.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/haste](https://github.com/microsoft/haste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
