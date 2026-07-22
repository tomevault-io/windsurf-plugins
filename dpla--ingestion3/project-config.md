---
trigger: always_on
description: Use this document when you are asked to **run ingests**, **report status**, or **handle errors** for the DPLA ingestion pipeline. This document provides shared reference (environment, notifications, error patterns). For ingest procedures, use the rules and skills linked in the Ingest workflows section.
---

# DPLA Ingestion3 – Agent guide

Use this document when you are asked to **run ingests**, **report status**, or **handle errors** for the DPLA ingestion pipeline. This document provides shared reference (environment, notifications, error patterns). For ingest procedures, use the rules and skills linked in the Ingest workflows section.

---

## Environment and build

**Before any of these:** building the JAR, running `./scripts/*`, running the orchestrator, running Python tools, or running AWS scripts that need project env — you must load the project environment. This is the **canonical checklist** for Cursor and Claude agents; .cursorrules and CLAUDE.md point here for the full list.

- **Load env:** From repo root run `source .env` so `JAVA_HOME`, `SLACK_WEBHOOK`, `DPLA_DATA`, `I3_CONF`, etc. are set. Scripts that source `common.sh` (e.g. `harvest.sh`, `ingest.sh`) automatically load `$I3_HOME/.env` when present, so `SLACK_WEBHOOK` is available for harvest-failure notifications even if you did not run `source .env` in the shell first.
- **Build the fat JAR:** From repo root run `source .env` then `sbt assembly`. Java 11+ is required (the codebase uses `java.net.http`). If you run `sbt assembly` without sourcing `.env`, the build may use the wrong JDK and fail (e.g. "package java.net.http does not exist"). Pipeline scripts (harvest.sh, ingest.sh, etc.) automatically run `sbt assembly` when the JAR is missing or when any Scala source is newer than the JAR, so the "harvest [hub]" skill uses current code without a separate build step.
- **Java:** 11+ required for build and run. Set `JAVA_HOME` in `.env` (see Java requirement section below).
- **Python:** Use `./venv/bin/python` or `source ./venv/bin/activate`; do not use system Python.
- **AWS:** Use `--profile dpla` for all `aws` commands.

**Why this checklist lives in AGENTS.md:** The full list of "before X do Y" rules is kept in one place so Cursor and Claude agents (and humans) follow the same process. .cursorrules and CLAUDE.md point here for the canonical checklist.

---

## Ingest workflows

| Task | Where to go |
|------|-------------|
| Run a single-hub ingest | [.claude/rules/ingestion.md](.claude/rules/ingestion.md) / dpla-run-ingest skill |
| Run orchestrator (parallel, scheduled) | [.claude/rules/orchestrator.md](.claude/rules/orchestrator.md) |
| Verify and notify on failure | [.claude/rules/notifications.md](.claude/rules/notifications.md) |
| Runbook index | [runbooks/README.md](runbooks/README.md) |

> Rules and skills are synced from `docs/ai-context/`. Edit there; run `./scripts/ai-context/sync.sh`.

---

## Notifications and errors (Slack or email tech@dp.la)

**Policy:** All errors and important status updates must be reported to **Slack** or **email tech@dp.la** (or both). Prefer Slack when a webhook is configured; otherwise ensure tech@dp.la is notified.

**Treat as errors (must notify):**
- Feed or endpoint unreachable (e.g. OAI/API timeout, connection failure).
- Harvest failure (script or orchestrator harvest step failed).
- Mapping/remap or enrichment failure.
- S3 sync failure.
- Anomaly detection halting sync (e.g. critical record-count or failure-rate change).
- Any pipeline step that exits non-zero or is reported as failed by the orchestrator.

**Treat as status (should notify):**
- Ingest run started (which hubs, run id).
- Ingest run completed (per-hub success/failure counts, draft emails path if applicable).
- Anomaly warnings (sync proceeded but unexpected changes).
- Failures requiring attention (list of failed hubs, link or path to failure report).

### Post stage failures to #tech-alerts

For any **pipeline stage failure**, ensure a notification is posted to **Slack #tech-alerts** (or, if Slack is unavailable, email tech@dp.la with the same information).

**Stage-specific failures to report:**

| Stage | What to report |
|-------|----------------|
| **Harvest** | Feed unreachable, timeout, or harvest script/orchestrator step failed. |
| **Mapping/remap** | Mapping or IngestRemap step failed (e.g. non-zero exit, no output). |
| **Sync** | S3 sync failed or was blocked. |
| **Anomaly** | Anomaly detection halted sync (critical threshold); optionally mention warning-level anomalies in #tech-alerts. |

**How it works:** When using the orchestrator (`python -m scheduler.orchestrator.main`), [scheduler/orchestrator/notifications.py](scheduler/orchestrator/notifications.py) sends a Slack failure alert for the run (run id, failed hubs, report path). Configure the Slack webhook so that message posts to **#tech-alerts**. When running scripts manually, post a short message to #tech-alerts (or email tech@dp.la) with: hub name, stage that failed, and a one-line error or path to logs/report.

---

## Where notifications are implemented

- **Orchestrator** ([scheduler/orchestrator/](scheduler/orchestrator/)): When running the automated pipeline (`python -m scheduler.orchestrator.main`), it uses `notifications.py` to send:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dpla/ingestion3](https://github.com/dpla/ingestion3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
