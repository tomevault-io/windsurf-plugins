---
trigger: always_on
description: > AI-powered serverless invoice extraction for restaurant partner reconciliation
---

# UberEats Invoice Processing Pipeline

> AI-powered serverless invoice extraction for restaurant partner reconciliation

---

## Project Context

**Business Problem:** 3 FTEs spend 80% of time on manual data entry from delivery platform invoices, causing R$45,000+ in reconciliation errors quarterly.

**Solution:** Cloud-native serverless pipeline using Gemini 2.0 Flash for document extraction with autonomous monitoring via CrewAI.

**Critical Deadline:** April 1, 2026 (Q2 financial close)

**Requirements:** See [notes/summary-requirements.md](notes/summary-requirements.md) for consolidated requirements from 6 planning meetings.

---

## Architecture Overview

```text
INGESTION          PROCESSING                              STORAGE
─────────          ──────────                              ───────

┌───────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ TIFF  │──▶│ TIFF→PNG │──▶│ CLASSIFY │──▶│ EXTRACT  │──▶│  WRITE   │──▶ BigQuery
│ (GCS) │   │          │   │          │   │ (Gemini) │   │          │
└───────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
    │           │              │              │              │
    └───────────┴──────────────┴──────────────┴──────────────┘
                          Pub/Sub (events)
                               │
                          ┌────┴────┐
                          │   DLQ   │ ◀── Failed messages
                          │Processor│
                          └─────────┘

OBSERVABILITY                              AUTONOMOUS OPS
─────────────                              ──────────────

┌───────────┐  ┌───────────┐  ┌───────────┐    ┌─────────┐  ┌───────────┐  ┌──────────┐
│ LangFuse  │  │Cloud Logs │  │ Metrics   │    │ TRIAGE  │─▶│ROOT CAUSE │─▶│ REPORTER │─▶ Slack
└───────────┘  └───────────┘  └───────────┘    └─────────┘  └───────────┘  └──────────┘
```

| Stage | Technology | Purpose |
| ----- | ---------- | ------- |
| Cloud | GCP | Primary infrastructure |
| Compute | Cloud Run Functions | Serverless event-driven functions |
| Messaging | Pub/Sub | Event-driven pipeline with DLQ |
| Storage | GCS | File storage (input, processed, archive) |
| Data Warehouse | BigQuery | Extracted invoice data |
| LLM | Gemini 2.0 Flash | Document extraction |
| LLM Fallback | OpenRouter | Backup provider |
| LLMOps | LangFuse | LLM observability |
| Validation | Pydantic v2 | Structured output validation |
| IaC | Terraform + Terragrunt | Infrastructure provisioning |
| Autonomous Ops | CrewAI | AI agents for monitoring |

---

## Project Structure

```text
btc-zero-prd-claude-code/
├── src/                           # Main source code
│   ├── __init__.py
│   └── invoice_extractor/         # Core extraction library
│       ├── cli.py                 # CLI interface
│       ├── extractor.py           # Extraction logic
│       ├── image_processor.py     # Image processing
│       ├── llm_gateway.py         # LLM abstraction
│       ├── models.py              # Pydantic models
│       ├── validator.py           # Validation logic
│       └── tests/                 # Unit & integration tests
│
├── functions/                     # Cloud Run Functions
│   └── gcp/v1/                    # GCP functions v1
│       ├── src/
│       │   ├── functions/         # 5 Cloud Run functions
│       │   │   ├── tiff_to_png/   # TIFF→PNG converter
│       │   │   ├── invoice_classifier/  # Document classification
│       │   │   ├── data_extractor/      # Gemini extraction
│       │   │   ├── bigquery_writer/     # BigQuery loader
│       │   │   └── dlq_processor/       # Dead Letter Queue handler
│       │   └── shared/            # Shared utilities
│       │       ├── adapters/      # GCS, Pub/Sub, BigQuery, LLM
│       │       ├── schemas/       # Pydantic models
│       │       └── utils/         # Logging, GCS utils
│       └── tests/                 # Function tests
│
├── gen/                           # Code generation tools
│   └── synthetic_invoice_gen/     # Synthetic test data generator
│       └── src/invoice_gen/       # Invoice generation library
│
├── tests/                         # Test suites
│   └── smoke/                     # End-to-end smoke tests
│       ├── cli.py                 # Smoke test CLI
│       ├── runner.py              # Test orchestrator
│       ├── stages/                # Pipeline test stages
│       │   ├── generate.py        # Generate test invoices
│       │   ├── upload.py          # Upload to GCS
│       │   ├── process.py         # Trigger processing
│       │   ├── validate.py        # Validate results
│       │   ├── bigquery.py        # Check BigQuery
│       │   └── logging.py         # Check logs
│       └── validators/            # Field validation
│
├── infra/                         # Infrastructure as Code
│   ├── modules/                   # Terraform modules
│   │   ├── bigquery/              # BigQuery dataset/tables
│   │   ├── cloud-run/             # Cloud Run functions
│   │   ├── gcs/                   # GCS buckets
│   │   ├── iam/                   # Service accounts & roles
│   │   ├── pubsub/                # Topics, subs, DLQ
│   │   └── secrets/               # Secret Manager
│   └── environments/              # Terragrunt environments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owshq-academy/btc-zero-prd-claude-code](https://github.com/owshq-academy/btc-zero-prd-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
