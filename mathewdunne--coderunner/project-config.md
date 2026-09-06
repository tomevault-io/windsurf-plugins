---
trigger: always_on
description: A browser-based IDE for learning FRC robot programming. Students write Java, click Run, and watch their robot simulate in real time with telemetry rendered by AdvantageScope Lite. Each student gets a per-student VSCodium container with bundled redhat.java and wpilibsuite.vscode-wpilib extensions for full VS Code editor features.
---

# CodeRunner — Repo Notes for Codex

## What This Is

A browser-based IDE for learning FRC robot programming. Students write Java, click Run, and watch their robot simulate in real time with telemetry rendered by AdvantageScope Lite. Each student gets a per-student VSCodium container with bundled redhat.java and wpilibsuite.vscode-wpilib extensions for full VS Code editor features.

Architecture and design details: [`docs/about/architecture.md`](./docs/about/architecture.md). Decision logs live in [`docs/decisions/`](./docs/decisions/).

## Stack Rule

All non-container code is **TypeScript on Bun**. Use Bun for package management, TypeScript script execution, and the control-plane runtime. Keep `tsc --noEmit`/project references for typechecking.

Inside the V2 code container, Java/Gradle/WPILib, VSCodium, `redhat.java`, and `wpilibsuite.vscode-wpilib` are the relevant stacks.

## Repo Layout

```text
apps/control/                  Bun control plane: HTTP, WS, sessions, orchestration
apps/control/src/app.ts          slim factory + top-level fetch dispatcher
apps/control/src/app/            response/asset/proxy/status helpers + admin, workspace, websocket route groups
apps/control/src/containers.ts   barrel re-exporting the public container surface
apps/control/src/containers/     Docker client, metadata, ports, lifecycle, and the LocalDockerRuntimeProvider class
apps/control/src/metrics.ts      Prometheus registry, metric handles, route-templating helpers
apps/control/src/metrics-collector.ts  15s Docker stats poller that writes per-container gauges
apps/web/                      React + Vite browser IDE shell
packages/contracts/            Shared API schemas, message types, and path rules
containers/code/               V2 merged VSCodium + sim container
containers/control/            Control-plane image: multi-stage build burying the emsdk/AdvantageScope compile, the coderunner dispatching entrypoint
catalog/                       Bundled (zero-config) lesson catalog: modules.json + modules/<id>/, baked into the code image
lessons-repo-root/             Staging for the standalone remote lessons repo (will move out of this repo); not used by the app build
scripts/                       TypeScript utility scripts run by Bun
patches/advantagescope/        Source-level AS Lite patches
docs/                          Site content (Docusaurus pages) + decision logs (docs/decisions/)
website/                       Docusaurus site config; docs/ is the content source
dashboards/                    Pre-built Grafana ops dashboard JSON (import into Grafana Cloud)
vendor/AdvantageScope/         Pinned upstream submodule
e2e/                           Playwright E2E tests (specs/ and fixtures/)
data/                          Runtime data, gitignored
```

## Current Status

- [x] V1-0 through V1-10: V1 complete (archived)
- [x] V2-0: editor spike accepted and recorded in `docs/decisions/011-v2-editor-spike.md`
- [x] V2-1: merged code container image
- [x] V2-2: authenticated editor proxy
- [x] V2-3: orchestrator merge and run-path migration
- [x] V2-4: web shell swap to hosted editor
- [x] V2-5: file API and contracts cleanup
- [x] V2-6: lifecycle, labels, and reconciliation
- [x] V2-7: acceptance pass

V2 is complete. The system uses per-student merged containers (`coderunner-workspace`) running VSCodium with bundled Java and WPILib extensions. The control plane proxies editor, run, and telemetry traffic through authenticated routes.

**Lessons & Modules (post-V2):** first-login template seeding is removed —
workspaces start empty and the student fills them via the topbar **Switch
Project** surface, which offers a lesson catalog plus a public GitHub team
import. The catalog has two sources behind one interface: a **bundled** `catalog/`
(zero-config, baked into the image) and a **remote** lessons repo when
`LESSONS_CATALOG_REPO` is set. Catalog loads are gitless (reset = re-load); team
imports keep `.git` for push. The per-import backup/restore flow was removed
(pure discard + git). See [`docs/lessons/overview.md`](./docs/lessons/overview.md)
and `docs/decisions/029-lessons-and-modules.md`.

**Containerized control plane (post-V2):** the control plane ships as a Docker
image (`containers/control/Dockerfile` → `ghcr.io/mathewdunne/coderunner-control`)
and is deployed with docker compose (`docker-compose.yml` base +
`docker-compose.prod.yml` for Caddy/Alloy; demo mode is `CODERUNNER_DEMO_MODE=1
docker compose up`, an env passthrough rather than an override file). It runs
the host Docker daemon over the bind-mounted socket and manages workspace
containers as siblings. The control container runs **non-root** as the data-dir
owner (image default `USER bun`; compose overrides via
`user: ${CODERUNNER_UID}:${CODERUNNER_GID}` with `group_add:
${CODERUNNER_DOCKER_GID}` for socket access), so `./data` stays host-owned, not
root-owned. Two modes via env: **port mode** (default;
`FRC_CONTAINER_NETWORK` unset) publishes loopback ports and is what

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mathewdunne/CodeRunner](https://github.com/mathewdunne/CodeRunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
