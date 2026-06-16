---
trigger: always_on
description: kube-llmops is a Kubernetes-native LLMOps platform using Umbrella Helm Charts.
---

# AGENTS.md — Project Knowledge for AI Assistants

## Project Overview
kube-llmops is a Kubernetes-native LLMOps platform using Umbrella Helm Charts.
Deploy, manage, monitor, and optimize LLM infrastructure with a single `helm install`
(or via the LLMPlatform CR managed by the included Kubernetes Operator).

## Key Commands

```bash
# Deploy (single-node with GPU + NodePort access) — direct Helm
NODE_IP=$(kubectl get node -o jsonpath='{.items[0].status.addresses[0].address}')
helm install kube-llmops charts/kube-llmops-stack \
  -f charts/kube-llmops-stack/values-single-node.yaml \
  --set global.nodePort.enabled=true \
  --set global.nodePort.host=$NODE_IP \
  --set global.hfToken=$HF_TOKEN

# Upgrade
helm upgrade kube-llmops charts/kube-llmops-stack \
  -f charts/kube-llmops-stack/values-single-node.yaml \
  --set global.hfToken=$HF_TOKEN --no-hooks

# Alternative: Operator-managed deployment via LLMPlatform CR
# 1) Build + push operator image (one-time; see operator/build.sh)
bash operator/build.sh
docker tag kube-llmops/operator:latest localhost:5000/kube-llmops/operator:latest
docker push localhost:5000/kube-llmops/operator:latest
# 2) Install operator chart (embeds the umbrella chart)
helm install kube-llmops-operator operator/charts/kube-llmops-operator
# 3) Apply an LLMPlatform CR (see operator/config/samples/llmplatform_full.yaml)
kubectl apply -f operator/config/samples/llmplatform_full.yaml

# IMPORTANT: After changing any subchart template, rebuild archives:
cd charts/kube-llmops-stack && rm -f charts/*.tgz Chart.lock && helm dependency update .

# Build model-loader image (first time only)
docker build -t kube-llmops/model-loader:latest images/model-loader/

# Build Headlamp plugin image (first time only)
docker build -t kube-llmops/headlamp-plugin:latest plugins/kube-llmops-portal/

# Run Playwright E2E tests
uv run tests/e2e/test_dify_model_provider.py
uv run tests/e2e/test_dify_rag_e2e.py

# Trigger Ragas evaluation
kubectl create job ragas-manual --from=cronjob/kube-llmops-ragas-eval

# Check smoke test
kubectl logs -l app.kubernetes.io/name=rag-smoke-test --tail=30

# Check quality gate
kubectl logs job/kube-llmops-quality-gate

# Run all Helm template tests (Phase 5 + finetune + module switches)
python -m pytest tests/helm/ -v

# Run finetune E2E tests (requires GPU cluster + Argo Workflows)
uv run tests/e2e/test_finetune_e2e.py

# Run Phase 5 Helm template tests
python -m pytest tests/helm/test_phase5_templates.py -v
```

## Architecture

```
┌─ Ingress (Traefik) / NodePort ───────────────────┐
│  *.llmops.local → litellm/grafana/langfuse/dify  │
│  or NODE_IP:304xx                                 │
├──────────────────────────────────────────────────┤
│ LiteLLM (Gateway:4000) → vLLM (LLM:8000)        │
│                        → SGLang (MoE/VLM:30000)  │
│                        → Chitu (Domestic:21002)   │
│                        → TEI (Embed:8080)         │
│                        → TEI (Rerank:8080)        │
│                        → llama.cpp (GGUF:8080)    │
│ Dify (RAG:5001/3000) → LiteLLM → pgvector        │
│ Langfuse (Trace:3000) ← LiteLLM callbacks         │
│ Prometheus:9090 + Pushgateway:9091 → Grafana:3000 │
│ Node Exporter:9100 + Kube State Metrics:8080      │
│ LLM-Guard (Security:8000)                         │
│ Keycloak (SSO:8080 / HTTPS:8443)                  │
│ Argo Workflows + LLaMA-Factory (Fine-tune)          │
│ MLflow (Experiment Tracking:5000)                    │
│ Headlamp (K8s UI:4466) + Portal Plugin             │
│ MinIO (S3:9000) + PostgreSQL:5432 + Harbor:80      │
│ kube-llmops-operator (LLMPlatform/ModelDeployment/ │
│                       FineTuneRun CRDs)             │
└──────────────────────────────────────────────────┘
```

## Key Features (v1.0.0)

### kubectl-llmops CLI
Kubectl plugin for imperative shortcuts — complement to the declarative Operator:
- Binary: `kubectl-llmops` (invoke as `kubectl llmops <command>` when on PATH)
- Source: `operator/cmd/kubectl-llmops/main.go` (impl: `operator/internal/cli/cmd/`)
- Build: `cd operator && make build-cli` (produces `bin/kubectl-llmops`) or `make install-cli` (to `$GOPATH/bin`)
- Global flags: `-n/--namespace`, `-o table|json|yaml|wide`, `--kubeconfig`, `--context`
- Top-level commands (model lifecycle):
  - `deploy <source>` — create ModelDeployment from HF source (engine auto-detected)
  - `list` / `status <name>` / `scale <name> --replicas N` / `delete <name>`
  - `canary <name> --target <new-source> --weight <percent>` / `promote` / `rollback`
- Developer UX:
  - `logs <name> [-f]` / `test <name> --prompt "..."` / `endpoint <name>`
  - `port-forward --service=gateway|grafana|langfuse|dify|minio`
  - `dashboard` (opens Grafana)
- Subcommand groups:
  - `finetune {create,list,status,logs,delete}` — drives FineTuneRun CR
  - `platform {init,status,update}` — drives LLMPlatform CR (incl. module toggles)
  - `rag {list-kb,create-kb,upload,delete-kb,query,eval}` — Dify API operations
- `migrate <helm-release>` — one-way conversion: existing Helm release → LLMPlatform + ModelDeployment CRs
- Integration:
  - CR operations (deploy/scale/canary/finetune/platform) go through K8s API → operator reconciles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaeaRuiW/kube-llmops](https://github.com/GaeaRuiW/kube-llmops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
