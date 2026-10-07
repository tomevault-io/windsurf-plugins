---
trigger: always_on
description: Portfolio project demonstrating production-grade AI infrastructure engineering: a scale-to-zero GPU inference platform built on Kubernetes. GPU nodes provision on demand when requests arrive and deprovision when idle — cost is $0 when the system is not in use.
---

# gpu-autoscale-inference

## Overview

Portfolio project demonstrating production-grade AI infrastructure engineering: a scale-to-zero GPU inference platform built on Kubernetes. GPU nodes provision on demand when requests arrive and deprovision when idle — cost is $0 when the system is not in use.

This project targets roles: **ML Platform Engineer, AI Infrastructure Engineer, LLM Systems Engineer**.

## The Core Concept

LLM infrastructure faces a hard tradeoff: GPUs must stay active for low latency, but should shut down when idle to minimize cost. This project solves it with two layers of autoscaling:

- **KEDA (pod autoscaler):** watches Redis queue depth → scales Worker + vLLM pods 0↔1
- **Cluster Autoscaler (node autoscaler):** sees pending GPU pod → provisions/deprovisions the GPU VM

Result: no requests → no pods → no GPU node → $0/hr.

## Tech Stack

| Layer | Tool |
|---|---|
| API Gateway | FastAPI |
| Request Queue + Result Store | Redis |
| Pod Autoscaler | KEDA |
| Node Autoscaler (cloud) | Cluster Autoscaler |
| Queue Consumer | custom Python worker |
| Inference Engine | vLLM (Qwen/Qwen2.5-1.5B-Instruct) |
| Orchestration | Kubernetes (raw Deployment + Service) |
| Observability | Prometheus + Grafana + NVIDIA dcgm-exporter |
| Load Testing | Locust |

## Architecture

```
User
 │
 ▼
API Gateway (FastAPI)  →  POST /generate  →  enqueue job  →  return {job_id}
                                                  │
                                             Redis Queue
                                                  │
                              KEDA monitors queue depth (threshold: 5)
                                                  │
                               queue > 5 → scale up Worker (0→1) + vLLM (0→1)
                                                  │
                              [cloud] Cluster Autoscaler provisions GPU node
                                                  │
                              vLLM loads Qwen2.5-1.5B (~5-10s)
                              readiness probe passes
                                                  │
                              Worker pulls job → POSTs to vLLM → writes result
                                                  │
                              Client polls GET /result/{job_id}
                                                  │
                              Queue drains → KEDA scales to 0
                              [cloud] Cluster Autoscaler removes GPU node
```

## Cluster Layout

```
KUBERNETES CLUSTER
├── Node A — CPU VM (always-on, cheap)
│   ├── Pod: gateway
│   ├── Pod: Redis
│   ├── Pod: KEDA
│   └── Pod: Cluster Autoscaler
│
└── Node B — GPU VM (provisions/deprovisions on demand)
    ├── Pod: vLLM        ← KEDA scales, uses nvidia.com/gpu: 1
    └── Pod: worker      ← KEDA scales, calls vLLM over HTTP
```

## API Contract

| Endpoint | Method | Response |
|---|---|---|
| `/generate` | POST (JSON body: `{prompt: str}`) | `{status: "queued", job_id: "..."}` |
| `/result/{job_id}` | GET | `{status: "pending"}` or `{status: "done", response: "..."}` or `{status: "error", message: "..."}` |
| `/health` | GET | `{status: "ok"}` |

All requests are fully async — `/generate` always returns a `job_id`, never blocks for inference.

## KEDA Scaling Rules

- Worker: `minReplicaCount: 0`, `maxReplicaCount: 2`, trigger: `inference_queue` length > 5
- vLLM: `minReplicaCount: 0`, `maxReplicaCount: 1`, same trigger
- Ratio: 2 workers share 1 vLLM instance via HTTP (vLLM handles concurrency via continuous batching)

## Key Implementation Details

- `job_queue.py` — Redis helpers (NOT `queue.py` — name collision with Python stdlib)
- `VLLM_URL` — environment variable in worker: `http://host.docker.internal:8000` (Phase 1) or `http://vllm:8000` (Phase 2)
- `MODEL_ID` — environment variable in worker + vLLM deployment: `Qwen/Qwen2.5-1.5B-Instruct` (model-agnostic, swappable)
- `wait_for_vllm()` in worker — retry loop on `/health` endpoint, handles model load delay
- Result store TTL: 5 minutes — failed jobs write `{status: error}`, never leave key empty
- vLLM readiness probe: `httpGet /health`, `initialDelaySeconds: 10`, `periodSeconds: 10`

## Development Phases

### Phase 1 — Local GPU (current starting point)

vLLM runs on the **host in Docker** (not inside k3d). Everything else runs inside k3d.

```bash
# Start vLLM on host (uses local 8GB GPU)
docker run --gpus all -p 8000:8000 --ipc=host \
  vllm/vllm-openai --model Qwen/Qwen2.5-1.5B-Instruct \
  --max-model-len 4096 --gpu-memory-utilization 0.8 --enforce-eager

# Start k3d cluster
k3d cluster create llm-gateway --port "8080:80@loadbalancer"

# Deploy everything except vLLM
kubectl apply -f k8s/

# Run load test
source .venv/bin/activate
locust -f loadtest/locustfile.py --host http://localhost:8080
```

Worker env var for Phase 1: `VLLM_URL=http://host.docker.internal:8000`

Phase 1 does NOT demonstrate node-level autoscaling (GPU is always physically present locally).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adityonugrohoid/gpu-autoscale-inference](https://github.com/adityonugrohoid/gpu-autoscale-inference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
