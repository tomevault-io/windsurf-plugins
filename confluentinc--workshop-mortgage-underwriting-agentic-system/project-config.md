---
trigger: always_on
description: End-to-end mortgage underwriting workflow on **Confluent Cloud** with **AI agents**. Streams mortgage applications and payment history, enriches them with credit data via CDC, and applies automated fraud/risk assessment plus decisioning using Confluent Streaming Agents powered by Claude on Amazon Bedrock.
---

# Project: Mortgage Underwriting Agentic System

## Overview

End-to-end mortgage underwriting workflow on **Confluent Cloud** with **AI agents**. Streams mortgage applications and payment history, enriches them with credit data via CDC, and applies automated fraud/risk assessment plus decisioning using Confluent Streaming Agents powered by Claude on Amazon Bedrock.

## Architecture

```
mortgage_applications ──► enriched_mortgage_applications ──► enriched_mortgage_with_payments ──► mortgage_validated_apps ──► mortgage_decisions
                                      ▲                                    ▲
PROD.public.applicant_credit_score ──┘          applicant_payment_summary ┘
                                                            ▲
                                                 payment_history ─────────┘
```

**Data flow**: Mortgage applications and payment history are produced to Kafka by the data generator. Credit scores are sourced from Postgres via CDC connector. The CDC table is interpreted by Flink as a versioned table (Debezium upsert mode with flattened columns). Flink SQL enriches mortgage apps with credit scores via a **temporal join** and payment history via a second temporal join. Two AI agents then process the enriched data — one for fraud/risk assessment, one for approval/rejection decisions and email notifications.

## Deployment Modes

| Mode | Directory | Datagen | Description |
|------|-----------|---------|-------------|
| **Workshop** | `terraform/workshop/` | Local Docker (`local-datagen`) | Instructor provides Postgres DB and Bedrock credentials. Participants manually create Flink SQL statements in Labs 1 & 2. |
| **Self-serve** | `terraform/self-serve/` | ECS Fargate (`ecs-datagen`) | Terraform provisions AWS infrastructure (EC2 Postgres, IAM for Bedrock) in addition to base. Participants still do labs manually. |
| **Demo** | `terraform/demo_mode/` | ECS Fargate (`ecs-datagen`) | Fully automated — provisions AWS infra, all Flink DDL statements deployed via Terraform. Data gen runs continuously. Designed for always-on operation. |

All three modes use `module.base` (`terraform/modules/base/`). Self-serve and demo additionally use `module.aws`, `module.ecs_datagen`, and `module.cdc_connector`. Workshop uses `module.local_datagen` and `module.cdc_connector`. Demo additionally uses `module.flink_statements`.

## Directory Structure

```
terraform/
├── modules/
│   ├── base/              # Core: env, cluster, topics, schemas, LLM model, MCP, webapp
│   ├── local-datagen/     # Local Docker datagen container — used by workshop only
│   ├── ecs-datagen/       # ECS Fargate datagen container — used by self-serve and demo_mode
│   ├── cdc-connector/     # Postgres CDC connector + ALTER statements — used by all modes
│   ├── flink-statements/  # 8 Flink DDL statements (CTAS, CREATE AGENT, CREATE TOOL) — used by demo_mode only
│   └── aws/               # AWS infra (EC2 Postgres, IAM for Bedrock) — used by self-serve and demo_mode
├── workshop/              # Entry point for workshop mode
├── self-serve/            # Entry point for self-serve mode
├── demo_mode/             # Entry point for demo mode
├── data-gen/              # Data generator Java app + Dockerfile
│   └── datagen-app/       # Maven project: DataGenerator.java
├── code/
│   └── FlinkTableAPI/     # Flink Table API Java app (alternative to SQL for enrichment)
└── schemas/
    └── avro/              # AVRO schemas for mortgage_applications and payment_history
webapp/                    # Flask webapp for submitting mortgage applications (port 5001)
lab1/                      # Lab 1 instructions: CDC connector + Flink SQL enrichment
lab2/                      # Lab 2 instructions: AI agents for risk assessment + decisioning
```

## Key Components

### Data Generator (`terraform/data-gen/datagen-app/`)
- **Image**: `ghcr.io/ahmedszamzam/datagen:latest` — single multi-arch image (amd64 + arm64) for all modes
- **Java 17**, Maven, uses Kafka Avro serializer + JavaFaker + PostgreSQL JDBC
- **Deployment**: Local Docker (workshop) or ECS Fargate (self-serve/demo). ECS auto-restarts on failure. Local Docker uses `--restart on-failure`.

#### Stage 1: Seed Postgres (immediate, no throttle)
- Creates `applicant_credit_score` table with columns: `applicant_id` (PK), `applicant_name`, `credit_score`, `credit_utilization`, `open_credit_accounts`, `total_credit_limit`, `public_records`, `updated_at` (TIMESTAMP, DEFAULT NOW())
- Inserts 702 rows with `updated_at` populated (critical for CDC watermark — NULL timestamps prevent watermark advancement):
  - 100 high credit (750-850 score, 0 public records)
  - 500 medium credit (501-750 score)
  - 100 low credit (300-500 score)
  - John Doe (`C-100000`) — score 800-850, 0 defaults, guaranteed high credit
  - Omar Soli (`C-200000`) — score 300-350, 5 defaults, guaranteed low credit

#### Stage 2: Historical Payments (immediate, no throttle)
- Preloads all applicants from Postgres into in-memory cache (thread-safe `CopyOnWriteArrayList`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [confluentinc/workshop-mortgage-underwriting-agentic-system](https://github.com/confluentinc/workshop-mortgage-underwriting-agentic-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
