---
trigger: always_on
description: Gubernator is a powerful "Goldilocks" orchestrator that combines the **simplicity of Docker Swarm** (native Compose support, easy cluster joining) with the **flexibility of Nomad** (task-based logic, labels for hardware/AI targeting).
---

# Gubernator (gbnt) - Project Blueprint & Roadmap

Gubernator is a powerful "Goldilocks" orchestrator that combines the **simplicity of Docker Swarm** (native Compose support, easy cluster joining) with the **flexibility of Nomad** (task-based logic, labels for hardware/AI targeting).

## 🏛 Technical Foundation

* **Language:** Go (Golang)
* **State:** SQLite (Centralized on Manager, with local cache on Workers for resilience)
* **API:** Secured REST (Port 4000)
* **Web UI:** Web Dashboard (Port 4001)
* **Observability:** OpenTelemetry + Prometheus, Swagger, Healthchecks (Port 4002)
* **Engine:** Docker Engine API interaction

## 🗺 Development Roadmap: The Road to Rome

The development is divided into "Campaigns" (Sprints):

### Phase 1: The Foundation (The City-State)
*Goal: A single node running the API and managing local containers via `gbnt`.*
* **Gubernator Core:** Setup the Go project structure and the SQLite schema for tracking "Legions" (Services) and "Centurions" (Nodes).
* **The Forum (API):** Implement the REST server on port 4000. Integrate **swag** for automatic Swagger UI generation.
* **Local CLI:** Build the initial `gbnt` binary to talk to the local API.
* **The Docker Bridge:** Logic to translate a service request into a `docker-api` container creation.

### Phase 2: The Legion (Clustering & Networking)
*Goal: Multi-host communication and node registration.*
* **Cluster Logic:** Implement `gbnt legion init` and `join`.
* **Node Registry:** Nodes must "phone home" to the Manager’s API to register status and system info.
* **Join Tokens:** Implementation of a simple JWT or secret-based handshake for `join-token`.
* **Heartbeat System:** Manager tracks node availability (Active/Pause/Drain).

### Phase 3: The Command (Compose & Labels)
*Goal: Deploying complex stacks and targeting specific hardware.*
* **Stack Parser:** Implement `gbnt stack deploy`. Uses a Go library to parse `docker-compose.yml`.
* **Label Engine:** Logic to read `deploy.placement.constraints` from the Compose file and match them against Node labels (e.g., `gpu=true`, `arch=arm64`).
* **Scheduler MVP:** A simple "Least Loaded" or "Spread" algorithm to decide which node gets which container.

### Phase 4: The Watchtowers (Observability & Health)
*Goal: Telemetry and self-healing.*
* **OpenTelemetry Integration:** Export metrics (CPU, RAM, Uptime) to port 4002.
* **Healthchecks:** The Manager polls the `/health` of containers. If one falls, the Governor restarts it.
* **Prometheus Scraper:** Ensure the 4002 output is formatted correctly for Prometheus discovery.

### Phase 6-8: The Senate Mandate & Security
*Goal: Complete API, CLI context management, and Asymmetric Security.*
* **Full CLI Parity:** Implementation of full CRUD for Stacks, Services, Nodes, and Tasks.
* **Security & Isolation:** Asymmetric architecture implementing Bearer tokens (`GBNT_API_TOKEN`) for Port 4000, Basic Auth for Port 4001, and exposing Port 4002 completely isolated for internal monitoring.
* **Remote Contexts:** CLI authentication via `~/.gbntctl/config` with `gbnt config use-context`.

## 🛠 Enhanced Features (The "Nomad-Hybrid" Touch)

1. **Binary Portability:** The `gbnt` binary acts as both the Manager (API + DB) and the Worker (Agent) to simplify deployment.
2. **State Persistence:** Workers maintain a local SQLite cache ("Draft" mode) to keep containers running even if connection to the Manager is lost.
3. **Label Naming Convention:** Roman prefix theme for hardware/AI labels:
   * `gbnt.node.role=worker`
   * `gbnt.node.gpu=nvidia`
   * `gbnt.node.zone=europe-1`
4. **Automatic Swagger:** Access `http://localhost:4002/swagger/index.html` for immediate documentation of the `gbnt` API.

## 📋 Initial Database Schema (SQLite)

| Table | Purpose |
| --- | --- |
| **Nodes** | ID, IP, Role (Manager/Worker), Status, Labels (JSON). |
| **Stacks** | Name, Raw Compose File, Deployment Date. |
| **Services** | ID, StackID, Image, Desired Replicas, Constraints. |
| **Tasks** | Individual container instances, NodeID assigned, Status (Running/Dead). |

## 🛡 First Coding Milestone (The "Veni" Sprint)

1. **Initialize Go project** with `go mod`.
2. **Setup framework:** Gin or Echo for the API and GORM for SQLite.
3. **First Endpoint:** `GET /v1/node/ls` (Returns the current host info).
4. **First CLI Command:** Build `gbnt node ls` which calls the endpoint and prints a table in the terminal.

## 👑 Phase 5: The Empire (Expansion Packs & Advanced Architecture)

To ensure Gubernator can handle real-world, production-ready deployments, the following advanced features will be integrated:

### 1. Ingress & Service Discovery (The Aqueducts)
* **CoreDNS Integration:** A minimal Gubernator node deployment will consist of **Gubernator + CoreDNS + Caddy**. CoreDNS will be deployed so that all Docker containers can resolve internal IPs via DNS. Gubernator will act as the source of truth, actively updating CoreDNS records as containers spin up or die.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mario-ezquerro/gubernator](https://github.com/mario-ezquerro/gubernator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
