---
trigger: always_on
description: Spec-driven AI-enhanced software development lifecycle platform built on OpenChoreo.
---

# ASDLC Platform

Spec-driven AI-enhanced software development lifecycle platform built on OpenChoreo.

## Project Structure

```
asdlc/
├── console/            → React frontend (Vite + Oxygen UI)
├── asdlc-service/      → Go BFF + GitHub webhook receiver
├── git-service/        → Go microservice for git ops (clone, commit, push, tag)
├── agents/             → TS agents service (Vercel AI SDK; BA, Architect, TaskGen, Wireframe)
├── remote-worker/      → TS one-shot runner image for `app-factory-coding-agent` ClusterWorkflow
├── ui-components/      → pnpm workspace: explorer, md-editor, excalidraw-editor + dsl
├── deployments/        → CANONICAL local setup — k3d cluster + Docker Compose stack
├── tests/              → E2E (Playwright) + API integration (vitest)
├── docs/design/        → Architecture and component design docs
└── requirements/       → User scenario specifications
```

## Project Documentation

### Design Docs — `docs/design/`
- `architecture.md` — Overall system architecture, service diagram, data ownership
- `console.md` — Frontend (React + Oxygen UI)
- `api-service.md` — BFF (Go, PostgreSQL, OC proxy)
- `agent-orchestrator.md` — agents-service (Vercel AI SDK)
- `git-integration.md` — Git provider integration
- `openchoreo-client.md` — OpenChoreo client layer
- `auth-and-runtime-config-refactor.md` — auth + runtime-config canonical spec (window._env_ flow, BFF-owned ReleaseBinding env-config.js, per-project Thunder OAuth client)
- `testing.md` — Testing strategy

Keep design docs current. They reflect the high-level architecture, not specific tasks.

### Requirements — `requirements/`
- `1-project-management.md` … `6-manage-observe.md` — User scenarios for each lifecycle stage.

Every feature must have a corresponding user scenario. Scenarios double as the basis for E2E tests.

## Local Setup (`deployments/`)

The `deployments/` directory is the canonical local setup — a k3d cluster (OpenChoreo + Thunder + OpenBao + ESO + kgateway) plus a Docker Compose stack for the long-lived ASDLC services. The coding-agent runs as one-shot pods on the WorkflowPlane.

### Bring-up

```bash
bash deployments/scripts/setup.sh    # one-shot: k3d cluster + OC + Thunder + observability + ASDLC infra
$EDITOR deployments/.env             # set ANTHROPIC_API_KEY + optional GITHUB_APP_*
bash deployments/scripts/start.sh    # start the Docker Compose stack (BFF, agents, git-service, console)
```

**Console:** http://localhost:8090 · **Login:** `admin` / `admin` (default Thunder admin in the `Administrators` group; see `deployments/single-cluster/values-thunder.yaml`).

### Services

| Service | Tech | Where | Port |
|---|---|---|---|
| `asdlc-console` | React + Oxygen + nginx | compose | 8090 |
| `asdlc-api` (BFF) | Go, GORM, PostgreSQL | compose | 9090 |
| `asdlc-git-service` | Go, GORM | compose | 3300 |
| `asdlc-agents-service` | TS, Vercel AI SDK | compose | 3400 |
| `asdlc-db` (PostgreSQL 16) | — | compose | 5433 (host) |
| `asdlc-smee-client` | smee.io → BFF webhook relay | compose | — |
| `thunder` | WSO2 Thunder IDP | k3d | 8080/8090 (`thunder.openchoreo.localhost`) |
| `openbao` | Vault fork | k3d | 8200 (host) |
| `app-factory-coding-agent` | TS Claude Agent SDK | k3d (one-shot pod per task) | n/a |

The coding-agent runner image is `docker.io/xlight05/app-factory-coding-agent-runner:latest` (linux/amd64, `imagePullPolicy: Always`).

### Daily cycle

- **Iterate on a service:** `cd deployments && docker compose up -d --build <name>` (e.g. `asdlc-api`)
- **Tail logs:** `docker compose logs -f <name>`
- **Stop everything:** `bash deployments/scripts/stop.sh`
- **Teardown cluster:** `bash deployments/scripts/teardown.sh`
- **Rebuild + push runner image:** `cd remote-worker && docker buildx build --platform linux/amd64 -t docker.io/xlight05/app-factory-coding-agent-runner:latest --push .`

### Adding env vars

Edit the service's `environment:` block in `deployments/docker-compose.yml` and `docker compose up -d --force-recreate <name>`. For values that also need to be in the k3d cluster (Thunder seeds, OC clients), edit the relevant file under `deployments/single-cluster/` and re-run the setup step.

### GitHub webhooks

The BFF's `/webhooks/github` is reached via a smee.io channel (`GITHUB_WEBHOOK_PROXY_URL` in `.env`) and the `asdlc-smee-client` compose service. Smee occasionally reconnects and drops in-flight events — redeliver from GitHub's "Recent Deliveries" panel when state seems stuck.

## Artifact Storage and Versioning

Specs and designs are stored as files under `specs/` inside each project's cloned git repo (not PostgreSQL):

- `specs/requirements/` — `requirements.md` (required) + optional `functional-requirements.md`, `non-functional-requirements.md`, `user-stories.md`, `wireframes.{dsl,excalidraw}`, `domain-model.{dsl,excalidraw}`.
- `specs/design/` — `design.md` (root) + `components/<name>/design.md` (YAML frontmatter: `type`, `language`, `dependsOn`, `buildpack`, `appPath`, `entrypoint`, optional `exposesAPI` (services), optional `callerIdentity` (webapps)) + `components/<name>/openapi.yaml` (services only).

The BFF reads/writes via `ArtifactStore`; commits go through `git-service`. `ComponentTask` + `ComponentConfig` live in PostgreSQL.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wso2/labs-agentic-engineer](https://github.com/wso2/labs-agentic-engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
