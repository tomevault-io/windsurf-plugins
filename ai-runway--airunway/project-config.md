---
trigger: always_on
description: **AI Runway** is a platform for deploying and managing machine learning models on Kubernetes. It provides a unified CRD abstraction (`ModelDeployment`) that works across multiple inference providers (KAITO, Dynamo, KubeRay, llm-d, Direct vLLM, etc.).
---

# AI Runway - Agent Instructions

## WHY: Project Purpose

**AI Runway** is a platform for deploying and managing machine learning models on Kubernetes. It provides a unified CRD abstraction (`ModelDeployment`) that works across multiple inference providers (KAITO, Dynamo, KubeRay, llm-d, Direct vLLM, etc.).

## WHAT: Tech Stack & Structure

**Stack**:
- **Controller**: Go + Kubebuilder (Kubernetes operator)
- **Web UI**: React 18 + TypeScript + Vite (frontend) | Bun + Hono + Zod (backend)

**Key directories**:
- `controller/` - Go-based Kubernetes controller (kubebuilder project)
  - `controller/api/v1alpha1/` - CRD type definitions
  - `controller/internal/controller/` - Reconciliation logic
  - `controller/internal/webhook/` - Validation webhooks
  - `controller/config/` - Kustomize manifests for CRDs/RBAC
- `frontend/src/` - React components, hooks, pages
- `backend/src/` - Hono app, providers, services
- `providers/` - Standalone provider controllers/shims (`dynamo`, `kaito`, `kuberay`, `llmd`, `vllm`); each renders `ModelDeployment` into its upstream resource. `providers/vllm` is the in-repo Direct vLLM provider (renders native `Deployment`+`Service`, selected via `provider.name: vllm`).
- `shared/types/` - Shared TypeScript definitions
- `plugins/headlamp/` - Headlamp dashboard plugin
- `docs/` - Detailed documentation (read as needed; also the source rendered on the website)
- `website/` - Docusaurus site published to https://ai-runway.github.io/airunway/. Reads from `docs/` via `docs.path: '../docs'` — write docs as plain GitHub-Flavored Markdown and they render in both places.

**Core pattern**: Provider abstraction via CRDs:
- `ModelDeployment` - Unified API for deploying ML models
- `InferenceProviderConfig` - Provider registration with capabilities and selection rules

**Headlamp plugin**: When working on `plugins/headlamp/`, read [plugins/headlamp/README.md](plugins/headlamp/README.md) for patterns and best practices. Key rules: use Headlamp's built-in components (`SectionBox`, `SimpleTable`, etc.), never bundle React, use `@airunway/shared` for types/API.

**UI language**: Assume the user is **not familiar with Kubernetes**. All user-facing text in the Web UI and Headlamp plugin must use plain, approachable language:
- Avoid Kubernetes-specific terms in labels, descriptions, and hints — use everyday equivalents users already understand
- Add `InfoHint` tooltips to explain technical fields in plain language
- Kubernetes-specific terms are fine in code comments, YAML previews, and backend validation messages — just not in user-facing UI text

## HOW: Development Commands

### Controller (Go)
```bash
make controller-build       # Build Go controller binary
make controller-test        # Run controller tests
make controller-run         # Run controller locally
make controller-generate    # Regenerate CRDs and deepcopy code
make controller-install     # Install CRDs into cluster
make controller-deploy      # Deploy controller to cluster
```

### Web UI (TypeScript)
```bash
bun install              # Install dependencies
bun run dev              # Start dev servers (frontend + backend)
bun run test             # Run all tests (frontend + backend)
make compile             # Build single binary to dist/
make compile-all         # Cross-compile for all platforms
```

**After editing controller `*_types.go` files:**
```bash
cd controller && make manifests generate
```

### Headlamp Plugin Commands

```bash
cd plugins/headlamp
bun install              # Install plugin dependencies
bun run build            # Build plugin
bun run start            # Development mode with auto-rebuild
bun run test             # Run plugin tests
make setup               # Install deps, build, and deploy to Headlamp
make dev                 # Build and deploy for development
```

### Website (Docusaurus)

```bash
cd website
bun install              # Install website dependencies
bun run start            # Local dev server with hot reload
bun run build            # Production build (must pass before merge)
bun run serve            # Serve the production build locally
```

Docs sources stay in `/docs/*.md` (single source of truth). When the build
warns about a broken link or MDX issue, fix the source markdown — the site is
configured with `markdown.format: 'detect'` so `.md` files are treated as
plain GFM, not MDX. Anything in `{curly braces}` or bare `<angle-tags>` in a
`.md` file will only be parsed as JSX if the file is renamed to `.mdx`.

**Always run `bun run test` after implementing functionality to verify both frontend and backend changes.**

**Always validate changes immediately after editing files:**
- After editing Go files: Run `go build ./...` and `go test ./...`
- After editing frontend/backend files: Check for TypeScript/syntax errors
- If errors are found: Fix them before proceeding
- Never hand back to the user with syntax or compile errors

## Continuous Review


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-runway/airunway](https://github.com/ai-runway/airunway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
