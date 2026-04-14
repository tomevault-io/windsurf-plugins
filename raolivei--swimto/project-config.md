---
trigger: always_on
description: **IMPORTANT**: This project is part of the `raolivei` workspace. Workspace-level configuration is managed in the `workspace-config` repository.
---

# SwimTO - Cursor Rules

## Workspace Configuration

**IMPORTANT**: This project is part of the `raolivei` workspace. Workspace-level configuration is managed in the `workspace-config` repository.

- **Port Assignments**: See `../workspace-config/ports/.env.ports` for assigned ports
  - Web: Port `5173` (Vite default)
  - API: Port `8000` (assigned to avoid conflicts)
  - PostgreSQL: Port `5432`, Redis: Port `6379`
- **Workspace Conventions**: See `../workspace-config/docs/PROJECT_CONVENTIONS.md`
- **Port Conflict Checker**: Run `../workspace-config/scripts/check-ports.sh` before starting services
- **Workspace Scripts**: Available in `../workspace-config/scripts/`

When starting services locally, always use the assigned ports from `workspace-config` to prevent conflicts with other projects.

## Project Overview

SwimTO aggregates and displays indoor community pool drop-in swim schedules for Toronto. Commercial/proprietary project.

**CLUSTER**: Deployed on the **`eldertree`** Raspberry Pi k3s cluster.

## Security Requirements

### ALWAYS USE HTTPS

- ✅ **Production**: ALL external-facing services MUST use HTTPS
- ✅ **OAuth/Authentication**: Google OAuth REQUIRES HTTPS (blocks private IPs and HTTP)
- ✅ **Development**:
  - Use `http://localhost:5173` for local desktop testing
  - Use ngrok/Cloudflare Tunnel for mobile/external testing: `https://abc123.ngrok-free.app`
- ✅ **Kubernetes Ingress**: All ingress resources MUST use TLS certificates
- ❌ **NEVER**: Use `http://192.168.x.x` or `http://10.x.x.x` for OAuth callbacks (Google blocks these)
- ❌ **NEVER**: Expose authentication endpoints over HTTP in production

**OAuth Redirect URIs**:

```
✅ http://localhost:5173/auth/callback        (local dev only)
✅ https://swimto.example.com/auth/callback   (production)
✅ https://abc123.ngrok-free.app/auth/callback (mobile testing)
❌ http://192.168.2.48:5173/auth/callback     (BLOCKED by Google)
```

## Project Structure

```
swimTO/
├── apps/
│   ├── api/              # FastAPI backend (Python)
│   └── web/              # React + Vite frontend (TypeScript)
├── data-pipeline/        # ETL and data discovery (Python)
├── k8s/                  # Kubernetes manifests
├── scripts/              # Utility scripts
└── docs/                 # Documentation
```

## Code Conventions

### Python (Backend API)

- Follow PEP 8
- Use type hints for all functions
- Write docstrings (Google style)
- Use FastAPI dependency injection
- SQLAlchemy for database models
- Alembic for migrations

### TypeScript (Frontend)

- Use TypeScript strictly (avoid `any`)
- Prefer functional components with hooks
- Use TanStack Query for data fetching
- Tailwind CSS for styling
- Follow React best practices

### Git Workflow

**CRITICAL**: Create different branches for different work. Do not mix up work - each branch should focus on a single feature, fix, or task. Keep branches focused and separate.

**Branch Naming:**

- `feature/api/<name>` - Backend features
- `feature/web/<name>` - Frontend features
- `feature/pipeline/<name>` - Data pipeline
- `fix/<name>` - Bug fixes
- `infra/k8s/<name>` - Kubernetes changes
- `docs/<name>` - Documentation

**Commit Messages:**

- Use conventional commits: `feat(api):`, `fix(web):`, `docs:`
- Scope indicates component: `api`, `web`, `pipeline`, `k8s`

**Version Consistency:**

- **Git tag versions must match Docker image tag versions** - When creating a release, ensure the git tag (e.g., `v1.2.3`) matches the Docker image tag used in deployment manifests and Helm charts

**Image Changes:**

- **Any changes to Docker images must always update CHANGELOG.md** - This includes changes to Dockerfiles, base images, image tags in deployment manifests, or any other image-related modifications

**CHANGELOG Requirements:**

**ALWAYS update CHANGELOG.md when:**

- ✅ Adding new features (user-facing or internal)
- ✅ Fixing bugs or issues
- ✅ Changing behavior, UI, or UX
- ✅ Modifying Docker images, Dockerfiles, or deployment configs
- ✅ Making breaking changes or deprecations
- ✅ Updating dependencies that affect functionality

**Format:** Follow [Keep a Changelog](https://keepachangelog.com/) format:

- Version header: `## [X.Y.Z] - YYYY-MM-DD`
- Categories: `Added`, `Changed`, `Fixed`, `Removed`, `Deprecated`, `Security`
- Each entry starts with a dash and describes the change clearly
- Link related PRs/issues when applicable

**Git Command Restrictions:**

- **Never run git commands at workspace root**: Always ensure you are in the project directory (`swimTO/`) before running any git commands. Never run git commands at `/Users/roliveira/WORKSPACE/raolivei`

### Kubernetes Manifests

- **Use Helm charts where applicable** - Prefer Helm charts over raw YAML manifests when suitable charts exist
- Namespace: `swimto`
- Resource naming: `swimto-<component>`
- Separate deployments: `api-deployment.yaml`, `web-deployment.yaml` (or use Helm charts)
- Use ConfigMaps for non-sensitive config
- Secrets stored in Vault (see `scripts/setup-vault-secrets.sh`)

### Docker Images & GitHub Container Registry (GHCR)

- **Image Locations**:
  - API: `ghcr.io/raolivei/swimto-api:<tag>`
  - Web: `ghcr.io/raolivei/swimto-web:<tag>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raolivei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
