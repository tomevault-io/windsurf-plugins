---
trigger: always_on
description: Bronze → Silver → Gold data pipeline orchestration. Multi-agent architecture with AWS integration.
---


# CLAUDE.md — Agentic Data Onboarding System

Bronze → Silver → Gold data pipeline orchestration. Multi-agent architecture with AWS integration.

## STOP — Human-in-the-Loop Gate (Phase 1 Discovery)

**DO NOT GENERATE ANY PIPELINE CODE, SCRIPTS, CONFIGS, OR DAGS UNTIL YOU HAVE EXPLICIT HUMAN ANSWERS FOR ALL ITEMS BELOW.**

This is a human-in-the-loop gate. The HUMAN provides the rules. The agent does NOT guess or infer them.

### Required Questions (ask ALL before proceeding)

**Identify the zone first**, then ask zone-targeted questions (see `.claude/rules/00-zone-questions.md`):

- **Bronze** → source path, credentials, ingestion pattern, retention
- **Silver** → PK, dedup strategy, null handling, business logic, transformations
- **Gold** → business outcome, KPIs, aggregation grain, schema choice, BI tool

**Always ask regardless of zone:**
1. **PII/compliance** → which columns are PII? GDPR/CCPA/HIPAA/SOX/PCI?
2. **Quality** → thresholds per dimension, critical vs warning rules
3. **Scheduling** → cron expression, dependencies, failure handling

**Auto-discover first** (schema, format, nulls, row count) — then ask only what you couldn't discover. Present findings before questions.

### Completion Checklist — ALL must have HUMAN-PROVIDED answers

```
[ ] Zone identified (Bronze, Silver, Gold, or all)
[ ] Zone-specific questions answered (see rules/00-zone-questions.md)
[ ] Transformation rules confirmed by user (derived columns, calculations, custom logic — NEVER skip this even if you think "none needed")
[ ] PII columns and compliance requirements confirmed by user
[ ] Quality thresholds explicitly stated (or user says "use defaults")
[ ] Schedule explicitly stated by user
[ ] Ontology collection preference confirmed (opt-in/opt-out for semantic layer enrichment via Ontology agent)
[ ] If ontology YES: use cases + consumers confirmed (NL→SQL, discovery, BI, ML, compliance — who uses it?)
```

**If ANY item is missing, ASK THE USER. Do not proceed.**

### NEVER do these

- NEVER guess dedup strategy from column names
- NEVER infer null handling from data observations
- NEVER assume quality thresholds without user stating them
- NEVER generate a schedule based on source frequency — ask
- NEVER assume PII columns from names alone — ask user to confirm
- NEVER skip the transformation question — even if you auto-derived type casts and PII masking, you MUST ask about derived columns, calculations, and custom business logic
- NEVER skip the ontology question — always ask whether the user wants semantic layer enrichment (opt-in)
- NEVER auto-generate ontology entities, relationships, or business terms without user confirmation — present what you discovered, then ASK

You MAY profile data and PRESENT observations, then MUST ask: "How would you like to handle these?"

See `.claude/rules/01-human-gate-examples.md` for correct/incorrect behavior examples.

## Security Rules (Non-Negotiable)

1. **No hardcoded secrets** — use Secrets Manager, Airflow Connections, or env vars
2. **No infrastructure details in code** — no account IDs, VPC IDs, bucket names in source
3. **Encryption** — AES-256 at rest (KMS), TLS 1.3 in transit
4. **PII detection mandatory** — all workloads run `shared/utils/pii_detection_and_tagging.py`, LF-Tags + TBAC for column-level access
5. **Bronze immutability** — NEVER modify Bronze zone after ingestion
6. **Quality gates block promotion** — no bypassing
7. **Least privilege IAM** — no wildcard actions or resources
8. **Audit logging** — who, what, when, where for all operations

## Agent Behavior Protocol

1. **Route First** → check `workloads/` for existing data (found/not-found/partial)
2. **Ask Before Acting** → MANDATORY GATE above (Phase 1) — human provides all rules
3. **Deduplicate** → scan `workloads/*/config/source.yaml` for overlap (Phase 2)
4. **Profile** → 5% sample via Glue Crawler + Athena, present to human (Phase 3)
5. **Test Gates** → every sub-agent writes + passes tests before proceeding (Phase 4)
6. **Present Plan** → get human approval before multi-file changes

## Key Files

| File | Read When |
|---|---|
| `SKILLS.md` | Before acting as any agent |
| `TOOL_ROUTING.md` | Selecting which MCP tool to use |
| `MCP_GUARDRAILS.md` | Per-phase deploy guardrails |
| `tool-registry/servers.yaml` | Canonical MCP server list (13 servers) |
| `docs/workflow-diagrams.md` | Visual diagrams of flow |

## Folder Convention

```
workloads/{name}/
├── config/    # source.yaml, semantic.yaml, transformations.yaml, quality_rules.yaml, schedule.yaml
├── scripts/   # extract/, transform/, quality/, load/
├── dags/      # {name}_dag.py
├── sql/       # bronze/, silver/, gold/
├── tests/     # unit/, integration/
├── logs/      # Pipeline execution traces (trace_events.jsonl, run_*/)
└── README.md

shared/
├── operators/   # Reusable Airflow operators
├── hooks/       # Reusable Airflow hooks
├── utils/       # quality_checks.py, pii_detection_and_tagging.py, etc.
├── templates/   # dag_template.py, config_template.yaml
└── sql/common/  # Cross-workload SQL
```

## Data Zones

| Zone | Mutability | Quality Gate | Format |
|---|---|---|---|
| Bronze | Immutable | None | Raw source format |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-Agentic-Ai-Data-Operations](https://github.com/aws-samples/sample-Agentic-Ai-Data-Operations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
