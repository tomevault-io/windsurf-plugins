---
trigger: always_on
description: Reference implementation for secure, autonomous Data Engineering pipelines using the Medallion Architecture (Bronze → Silver → Gold) on local Delta Lake.
---

# Project Antigravity — Claude Code Instructions

Reference implementation for secure, autonomous Data Engineering pipelines using the Medallion Architecture (Bronze → Silver → Gold) on local Delta Lake.

## Architecture Overview

- **Bronze**: Raw, immutable, append-only ingestion with in-memory sanitization barrier
- **Silver**: Deduplicated, type-enforced, cleaned data via Delta MERGE
- **Gold**: Aggregated Star Schema facts and dimensions for business reporting

Agents handle implementation; these rules define the governance they must follow.

---

## Multi-Agent Team

Project Antigravity operates as a coordinated team of four specialized agents. The orchestrating Claude session selects agents based on their `description` field in `.claude/agents/`.

### Roles and Responsibilities

| Agent | Model | Tools | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| **data-architect** | Sonnet | Read, Glob, Grep | Schema design, dimensional modeling, lineage mapping, layer contracts |
| **data-steward** | Sonnet | Read, Glob, Grep, Bash | Compliance (GDPR/CCPA), security audits (SEC-01..SEC-06), data dictionary, access governance |
| **data-engineer** | Sonnet | Read, Glob, Grep, Bash, Write, Edit | Pipeline implementation (Bronze/Silver/Gold), environment setup, testing |
| **data-analyst** | Haiku | Read, Glob, Grep, Bash | DQ validation, metric verification, anomaly detection, quarantine reporting |

### Coordination Workflow

```
data-architect → data-steward → data-engineer → data-analyst
```

1. **Architect designs**: Produces a typed schema specification, lineage mapping, and Kimball dimensional model. Enforces GL-01..GL-04, BZ-03/BZ-06, SL-01/SL-04 at design time.
2. **Steward reviews**: Validates for GDPR/CCPA compliance, classifies data sensitivity (GL-07 mart assignment, GL-08 RLS), acts as approval gate for Gold schema changes (GL-10). Implementation does not begin until Steward approves.
3. **Engineer implements**: Writes and runs the pipeline following the approved design. Applies SEC rules during Bronze ingestion, CP rules throughout, and OPS-01 DAG ordering (Bronze → Silver → Gold, blocking on failure). Notifies Analyst on completion.
4. **Analyst validates**: Runs DQ gates (SL-04, GL-02, GL-03) against pipeline outputs and produces a structured validation report. Escalates to Steward (compliance issues) or Architect (structural/model issues).

### Tool Isolation Rationale

- **Architect**: Read-only — design must not accidentally mutate data or code.
- **Steward**: Read + Bash for audit tooling (Semgrep, Delta table inspection); no Write/Edit — violations are handed to Engineer for remediation.
- **Engineer**: Full access — sole implementation agent.
- **Analyst**: Read + Bash for queries and test execution; no Write/Edit — validation must not alter the data being validated. Haiku model is cost-appropriate for repetitive query-heavy checks.

---

## Security Policy

### Three Unshakeable Pillars
1. **Zero-Trust Ingestion** — Assume all incoming data is toxic (PII/Secrets) until proven otherwise.
2. **Immutable Audit** — Never delete history without a trace (Delta Time Travel).
3. **Ephemeral Secrets** — Agents never "know" credentials; they are injected only at runtime.

### Rule SEC-01: Secret Injection (12-Factor)
No agent shall ever possess a config file containing credentials. Use `os.environ['SRC_{NAME}_PASS']` strictly.

| Secret Type | Allowed | Prohibited |
| :--- | :--- | :--- |
| DB Passwords | `os.environ['SRC_DB_PASS']` | Hardcoded strings, `.env` files in code |
| AWS/Cloud Keys | IAM Role / Instance Profile | Long-lived Access Keys in code |
| API Tokens | Secrets Manager Reference | Plain text in `config.yaml` |

### Rule SEC-02: The Sanitization Barrier (Bronze — "No-Write" Policy)
Strict order of operations in the Bronze Agent:
1. **Fetch** — Pull data from source to **RAM only**
2. **Sanitize** — Apply regex masking in RAM
3. **Write** — Only then write to Delta Lake

Writing raw unmasked data to any disk location (including `/tmp/debug.json`) is a **Severity 1 Security Incident**.

### Rule SEC-03: Toxic Data Classification

| Category | Pattern | Action | Result |
| :--- | :--- | :--- | :--- |
| Credit Card (PAN) | `\b(?:\d[ -]*?){13,16}\b` | Mask (last 4 visible) | `XXXX-XXXX-XXXX-1234` |
| Passwords | Keys matching `pass`, `pwd`, `secret` | Drop or Hash | `[REDACTED]` |
| API Bearer Tokens | `Bearer\s+[a-zA-Z0-9\-\._~\+\/]+=*` | Redact | `Bearer *****` |

### Rule SEC-04: Storage Encryption
- **At Rest**: All Delta Tables must use Server-Side Encryption (SSE-S3 or KMS).
- **In Transit**: Enforce `sslmode=require` (Postgres) or `useSSL=true` (MySQL).

### Rule SEC-05: Right to be Forgotten (GDPR/CCPA)
Deletions are processed in Silver via Delta's `DELETE` command. A `VACUUM` must run within 7 days to physically purge Parquet files.

### Rule SEC-06: Security Envelope Columns
Every record must carry:
- `_ag_ingest_timestamp` — When was this data admitted?
- `_ag_source_system` — Who provided this data?
- `_ag_batch_id` — Which job run is responsible?

### Incident Response
1. **Stop the Bleeding**: Revoke and rotate the compromised credential immediately.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scardoso-lu/google-antigravity-settings](https://github.com/scardoso-lu/google-antigravity-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
