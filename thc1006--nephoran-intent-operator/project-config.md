---
trigger: always_on
description: > Single source of truth for Claude Code. Describes the **actual** system as built and deployed.
---

# CLAUDE.md — Nephoran Intent Operator

> Single source of truth for Claude Code. Describes the **actual** system as built and deployed.
> Last verified: 2026-03-05 against live cluster `thc1006-ubuntu-22` (K8s v1.35.1).

---

## 0. Hard Rules

1. **No direct kubectl from LLM** — LLM produces a schema-constrained IntentPlan JSON only. Never raw YAML, never direct apply.
2. **Everything is GitOps** — desired state changes go through Git → Porch/ConfigSync → cluster. No imperative mutations.
3. **Guardrails first** — validate plan (JSON Schema) → validate KRM (OPA + kubeconform + dry-run) → human review for sensitive resources (RBAC, CRDs, privileged, hostNetwork, cluster-scoped).
4. **Upstream first** — prefer O-RAN SC official images for RIC xApps and simulators.
5. **Identity by labels** — scale-out tracking uses resource names + labels, never Pod name suffixes. StatefulSet only when stable pod identity is required.

---

## 1. Project Identity

| Field | Value |
|---|---|
| Module | `github.com/thc1006/nephoran-intent-operator` |
| Go version | 1.26.0 |
| Goal | Intent-driven O-RAN lifecycle management: NL intent → LLM plan → KRM/kpt packages → GitOps → Porch/ConfigSync → workload cluster |
| Target HW | Single workstation, NVIDIA RTX 5080 (16 GB VRAM), 32 GB RAM, Ubuntu 22.04 |
| K8s | kubeadm v1.35.1 single-node, containerd 2.2.1 (no Docker daemon) |
| Standards | 3GPP TS 28.541 · O-RAN WG6/WG10 · TMF 921 (Intent API subset) |

---

## 2. Live Platform Status

### 2.1 Deployed Components

| Component | Namespace | Status | Notes |
|---|---|---|---|
| **Free5GC v3.3.0** | `free5gc` | Running | AMF, SMF, UPF×2, UDM, UDR, AUSF, NRF, NSSF, PCF, WebUI, MongoDB 8.0, UERANSIM gNB+UE |
| **O-RAN SC Near-RT RIC** | `ricplt` | Running | 14 Helm releases: A1, E2, O1, RTmgr, SubMgr, AlarmMgr, VES, Kong, Prometheus, DBAAS(Redis) |
| **RIC xApps** | `ricxapp` | Partial | ricxapp-scaling ✅, e2-test-client ✅, KPIMON scaled-to-0, srsran-gnb scaled-to-0 |
| **Nephoran Operator** | `nephoran-system` | Running | controller-manager (watches `intent.nephoran.com/v1alpha1` NetworkIntent CRD) |
| **Nephoran Web UI** | `nephoran-system` | Running | NodePort 30090 (K8s Dashboard style), old UI at 30081 |
| **Intent Ingest** | `nephoran-intent` | Running | HTTP service accepting NL text, calls Ollama for LLM processing |
| **Conductor Loop** | `conductor-loop` | Running | File-watching orchestration pipeline |
| **Porch v3.0.0** | `porch-system` | Running | porch-server, porch-controllers, function-runner ×2 |
| **ConfigSync** | `config-management-system` | Running | root-reconciler syncing `nephoran-packages` repo |
| **Ollama** | `ollama` + systemd | Running | llama3.1:latest, CPU-only (no GPU passthrough via DRA yet) |
| **RAG Service** | `rag-service` | Running | FastAPI uvicorn on host port 8000, connects to Ollama |
| **Weaviate v1.34.0** | `weaviate` | Running | Vector DB for RAG knowledge base |
| **Gitea** | `gitea` | Running | Local Git server, SSH NodePort 30022 |
| **Prometheus + Grafana** | `monitoring` | Running | kube-prometheus-stack, Grafana NodePort 30300 |
| **GPU Operator** | `gpu-operator` | Running | v25.10.1, DRA mode (not traditional device-plugin) |

### 2.2 CRD: NetworkIntent

```yaml
apiVersion: intent.nephoran.com/v1alpha1
kind: NetworkIntent
```
- Validated CRD at `config/crd/intent/intent.nephoran.com_networkintents.yaml`
- 42 NetworkIntent resources currently in cluster (default, free5gc, ran-a, ricxapp namespaces)
- 10 Porch PackageRevisions in catalog (5 Free5GC + 4 O-RAN + 1 hello-nephoran)

### 2.3 Known Issues

- **RIC ImagePullSecret missing**: `secret-nexus3-o-ran-sc-org-10002-o-ran-sc` not found; images run from local cache, but rebuild/upgrade will fail
- **GPU DRA mode**: K8s Pods cannot request `nvidia.com/gpu` via traditional resource requests; need DRA ResourceClaim for GPU workloads
- **Ollama dual deployment**: both in-cluster Pod and host systemd service running (~980 Mi total)
- **MongoDB readiness**: occasional 5s probe timeout (non-critical, self-recovers)

---

## 3. Repository Structure (Actual)

```
nephoran-intent-operator/
├── CLAUDE.md                     # THIS FILE
├── go.mod / go.sum               # Go 1.26, 120+ direct dependencies
├── Dockerfile                    # Multi-target: manager, conductor-loop, intent-ingest, llm-processor, etc.
│
├── llm_nephio_oran/              # Python — M1 MVP (intentctl CLI + intentd API + planner + validator)
│   ├── intentctl.py              # CLI: typer + rich, create/push commands
│   ├── intentd/app.py            # FastAPI TMF921 subset API server
│   ├── models.py                 # Pydantic IntentPlan, Action, SliceSpec, Policy, Guardrails
│   ├── planner/stub_planner.py   # Deterministic stub planner (to be replaced by local LLM)
│   ├── generator/kpt_generator.py # Stub kpt package generator
│   ├── validators/schema_validate.py # JSON Schema validator (Draft 2020-12)
│   ├── gitops/pr_stub.py         # PR automation stub
│   └── observability/metrics_stub.py # Prometheus metrics stub
│
├── src/                          # Python/TS microservices (for containerized deployment)
│   ├── llm-gateway/              # FastAPI: /generate, /hydrate, /analyze (wraps vLLM/Ollama)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thc1006/nephoran-intent-operator](https://github.com/thc1006/nephoran-intent-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
