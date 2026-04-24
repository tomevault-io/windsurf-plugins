---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Forkspacer Operator UI** - a React-based web interface for managing Forkspacer Kubernetes operator resources. It's part of a three-component ecosystem:

1. **Forkspacer Operator** - Core Kubernetes operator managing custom resources
2. **API Server** - Backend REST API (default: http://localhost:8421)
3. **Operator UI** (this repo) - Frontend dashboard served via Nginx

The UI communicates with the API Server to manage **Workspaces** and **Modules** (Helm charts, databases, services).

## Development Commands

### Local Development
```bash
# Install dependencies
npm install

# Start dev server (connects to localhost:8421 by default)
make dev

# Or manually with custom API URL
REACT_APP_API_BASE_URL=http://localhost:8421 npm start
```

### Testing
```bash
# Run tests in watch mode
npm test

# Run tests once (for CI)
npm test -- --watchAll=false

# Run tests with coverage
npm test -- --coverage --watchAll=false
```

### Building
```bash
# Production build
npm run build

# Docker build (production - uses relative /api/v1 paths)
docker build -t operator-ui:local .

# Docker build with custom API URL (for standalone deployments)
docker build --build-arg REACT_APP_API_BASE_URL=http://your-api:8421 -t operator-ui:local .
```

### Releases
```bash
# Create and push a version tag to trigger automated release
git tag v0.1.0
git push origin v0.1.0
# This triggers .github/workflows/release.yml which:
# 1. Automatically updates helm/Chart.yaml and helm/values.yaml versions (via make update-version)
# 2. Builds and pushes Docker image to ghcr.io with version tag
# 3. Packages and publishes Helm chart to GitHub Pages
# 4. Notifies parent forkspacer/forkspacer repo of new version
```

## Architecture

### Component Structure
```
src/
├── components/           # React components
│   ├── WorkspaceList.tsx        # Main workspace listing & management
│   ├── WorkspaceCard.tsx        # Individual workspace card UI
│   ├── WorkspaceDetailsModal.tsx # Workspace detail view
│   ├── CreateWorkspaceModal.tsx  # Workspace creation form
│   ├── ModuleCatalog.tsx        # Module catalog browser with grid/search
│   ├── ModuleCatalogCard.tsx    # Individual module card in catalog
│   ├── RepositoryManager.tsx    # Repository add/remove/enable UI
│   ├── SearchAndFilter.tsx      # Search/filter controls
│   ├── CustomCheckbox.tsx       # Reusable checkbox component
│   └── Toast.tsx                # Toast notification system
├── services/
│   ├── api.ts                   # API client (backend communication)
│   └── repositoryService.ts     # Module repository management
├── types/
│   ├── workspace.ts     # Workspace-related TypeScript types
│   ├── module.ts        # Module-related TypeScript types
│   └── catalog.ts       # Module catalog types
└── App.tsx              # Root component with tab navigation
```

### API Communication

**API Base URL Configuration:**
- Build-time: Set `REACT_APP_API_BASE_URL` env var or Docker build arg
- Default (production): Uses relative path `/api/v1` (assumes ingress/nginx proxy)
- Development: `make dev` sets to `http://localhost:8421`

**API Service** (`src/services/api.ts`):
- Singleton class `ApiService` exported as `apiService`
- All backend communication goes through this service
- Handles error parsing and API response format (`{success: {...}, error: {...}}`)

**Key API Methods:**
- Workspaces: `listWorkspaces()`, `createWorkspace()`, `deleteWorkspace()`, `updateWorkspace()`, `hibernateWorkspace()`, `wakeWorkspace()`
- Modules: `listModules()`, `createModule()`, `updateModule()`, `deleteModule()`
- Module Catalog: `fetchModuleCatalog(catalogUrl)` - Fetches module catalog from GitHub (default: https://raw.githubusercontent.com/forkspacer/modules/main/index.json)

### Deployment Architecture

**Production Setup (with Nginx proxy):**
```
User Request → Nginx (port 80)
                ├─> /api/v1/* → API Server (proxied via nginx.conf.template)
                └─> /* → React static files
```

The nginx configuration (`nginx.conf.template`) uses envsubst to inject `API_SERVER_URL` at runtime via `docker-entrypoint.sh`.

**Build Arguments:**
- `REACT_APP_API_BASE_URL`: API endpoint for React app
  - Empty/unset: Uses relative `/api/v1` (expects Nginx proxy)
  - Set: Uses absolute URL (e.g., `http://api-server:8421`)

## Key Concepts

### Workspaces
Represents a logical environment/namespace in the operator. Fields:
- `name`, `namespace`: Kubernetes identifiers
- `phase`: Current state (e.g., "Ready", "Pending")
- `type`: Connection type ("in-cluster" or "kubeconfig")
- `hibernated`: Whether workspace is hibernated
- `message`: Status message

### Modules
Deployable components (typically Helm charts) managed by the operator. Fields:
- `name`, `namespace`: Kubernetes identifiers
- `phase`: ModulePhase enum ("ready" | "installing" | "uninstalling" | "sleeping" | "sleeped" | "resuming" | "failed")
- `workspace`: Reference to parent workspace
- `source`: Module definition (raw YAML or HTTP URL)
- `config`: Key-value configuration parameters

Modules define:
- Dynamic form fields for configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forkspacer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
