---
trigger: always_on
description: AI-powered incident response SaaS framework. Plug into any cloud, any git provider, any LLM. Run `./setup_demo.sh` to go from zero to a running dashboard in one command.
---

# Sentinel Framework — CLAUDE.md

AI-powered incident response SaaS framework. Plug into any cloud, any git provider, any LLM. Run `./setup_demo.sh` to go from zero to a running dashboard in one command.

## Repository layout

```
sentinel/                  Core Python package (pip-installable as sentinel-framework)
  config/                  YAML config loader → typed dataclasses (SentinelConfig)
  core/                    Severity enum, Incident dataclass, PR review logic
  providers/
    base/                  Abstract base classes: BaseLLMProvider, BaseCloudProvider,
                           BaseGitProvider, BaseAlertingProvider
    llm/                   kserve.py · ollama.py · openai.py · anthropic.py · gemini.py · fallback.py
    cloud/                 aws.py · gcp.py
    git/                   github.py · gitlab.py
    alerting/              slack.py · pagerduty.py
  rag/                     Codebase indexer → embedder → pgvector store → similarity query
  registry.py              ProviderRegistry.from_config() — single wiring point

services/
  dashboard/               FastAPI REST API (port 8501) + single-page UI (Tailwind + vanilla JS)
    api.py                 All endpoints, RCA pipeline, LLM chain, Slack notifications
    code_context_builder.py  Live file scanner — extracts real buggy code at true line numbers
    ui/index.html          Single-page dashboard (no build step)
  kserve-local/            Local KServe V2 bridge → Ollama (port 8081)
  cloudwatch-alarm-receiver/  Lambda: CloudWatch Alarm → SNS → Sentinel incident (with AI RCA)
  log-analyzer/            Kinesis consumer: rule-based + ML severity classification
  root-cause-agent/        Step Functions handler: 5-step LLM RCA pipeline
  validator/               Validates alerts against CloudWatch / metrics signals
  pr-security-agent/       Scans PRs for secrets, vuln deps, OWASP issues
  loki-bridge/             Lambda: AlertManager webhook → Kinesis (for Loki/Grafana integration)
  shared/                  aws_clients.py — shared boto3 factory used by all Lambdas

infra/                     Terraform — all AWS resources
  main.tf                  Module wiring
  variables.tf             All input variables (including sentinel_dashboard_url)
  cloudwatch_alarm_receiver.tf  Lambda + IAM + SNS topic + subscription
  modules/                 s3 · dynamodb · kinesis · sqs · rds · elasticache · lambda
  step-functions/          Step Functions state machine JSON

helm/sentinel/             Helm chart — deploys dashboard + bridge to any K8s cluster
ml-core/                   KServe InferenceService YAML, MLflow training pipeline
observability/             Prometheus values, Grafana dashboard JSON, Loki alert rules, Jaeger values
target/                    Realistic buggy microservices used by code_context_builder for RCA demos
  services/auth/           db.py (pool bugs), token_cache.py (tz mismatch)
  services/payments/       gateway.py (missing timeout)
  services/search/         indexer.py (missing batching)
tests/                     pytest suite
scripts/                   bootstrap_floci.py, e2e_test.py, rewrite_history.py
```

## Running the demo locally

```bash
./setup_demo.sh          # starts everything: Floci + Ollama + KServe bridge + dashboard
# then open http://localhost:8501
```

The script is idempotent — re-run safely. It:
1. Checks prereqs (Python 3.10+, pip, curl, Docker)
2. Installs Python deps
3. Starts Floci (local AWS emulator) at `http://localhost:4566`
4. Starts Ollama daemon, picks best available instruction model (prefers `phi3:mini`), starts KServe bridge on port 8081
5. Creates Kinesis streams / SQS queues / DynamoDB tables in Floci
6. Seeds realistic demo incidents + validation records
7. Starts FastAPI dashboard on port 8501

To skip Ollama and use fallback RCA text: just don't have Ollama installed — it degrades gracefully.

## Running components individually

```bash
# Dashboard only (needs Floci running)
./run_dashboard.sh

# Manual dashboard start
PYTHONPATH=. python3 -m uvicorn services.dashboard.api:app --port 8501 --host 0.0.0.0

# KServe bridge only (needs Ollama running)
OLLAMA_MODEL=phi3:mini uvicorn server:app --app-dir services/kserve-local --host 0.0.0.0 --port 8081

# Bootstrap Floci resources only
python scripts/bootstrap_floci.py

# End-to-end test suite (needs Floci running)
python scripts/e2e_test.py

# Unit tests
pytest tests/
```

## Key environment variables

### LLM providers (dashboard)

| Variable | Default | Purpose |
|---|---|---|
| `KSERVE_ENDPOINT` | `http://localhost:8081` | KServe/Ollama bridge URL (always tried first) |
| `KSERVE_MODEL` | `llama3.2:1b` | Model name forwarded to Ollama |
| `GEMINI_API_KEY` | — | Enables Gemini in the fallback chain |
| `GEMINI_MODEL` | `gemini-1.5-flash` | Gemini model |
| `OPENAI_API_KEY` | — | Enables OpenAI in the fallback chain |
| `OPENAI_MODEL` | `gpt-4o-mini` | OpenAI model |
| `ANTHROPIC_API_KEY` | — | Enables Anthropic in the fallback chain |
| `ANTHROPIC_MODEL` | `claude-haiku-4-5-20251001` | Anthropic model |

### Alerting

| Variable | Default | Purpose |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VishalVinayRam/sentinel-framework](https://github.com/VishalVinayRam/sentinel-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
