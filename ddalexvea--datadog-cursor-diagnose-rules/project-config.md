---
trigger: always_on
description: Rules for troubleshooting DBM and APM correlation issues
---


# DBM and APM Correlation Troubleshooting

## Overview

APM–DBM correlation connects database spans from APM traces with query samples from Database Monitoring, enabling features like viewing database info in APM and APM data in DBM.

**Reference:** [Troubleshooting DBM and APM Correlation](https://datadoghq.atlassian.net/wiki/spaces/TS/pages/3183772859)

## Items to Collect

Before troubleshooting, obtain:

1. **APM trace link** containing a database span for the query in question
2. **DBM link** for the same exact query:
   - (ideally) A Query Sample
   - An Explain Plan
   - Confirmation whether Query Metrics exist for that query
3. **Note:** The DBM and APM query text should be exactly the same (small cosmetic differences like spaces are okay)

## Key Questions Checklist

### Question 1: Do you have a Query Sample or Explain Plan?

- **No** → No DBM sample/plan exists for this query → Go to **Bucket A**
- **Explain Plan only** → DBM has query metrics but not samples (common for SQL Server) → Go to **Bucket A/D**
- **Query Sample exists** → Go to Question 2

### Question 2: If you have a Query Sample, are there comments in `db.metadata.comments`?

Search DBM samples with:
```
@db.metadata.comments:* OR @trace.caller.service:*
```

Or open the Query sample → Event Attributes → `db.metadata.comments`

When comments are present, you should see fields like `dddbs`, `ddps`, etc.

Also check `@trace.span.resource_hash` for comparison with the APM span.

- **No comments** → Go to **Bucket B** or **Bucket D**
- **Comments present** → Go to Question 3

### Question 3: Do you have a trace with a database span, and what does the hidden metadata show?

**To view hidden span metadata (internal only):**

1. Click on the database span in the trace
2. Append `&config_trace_show_hidden_metadata=true` to the URL

⚠️ **DO NOT SHARE TRACE LINKS WITH THIS INTERNAL PARAMETER WITH CUSTOMERS**

**From the database span, record:**
- `dbm_trace_injected` → whether the tracer injected DBM comments
- `_dd.agent_version` → Agent version
- `_dd.tracer_version` → Tracer version
- `component` → DB library being traced
- `language` → application language
- `_dd.query_signature` → used for statement-based correlation

## Root Cause Buckets

---

### Bucket A: No Query Sample or Explain Plan Exists

**Scenario:** Question 1 = "No" (no sample or plan at all)

**Likely Causes:**
- Query is too infrequent or too fast → DBM doesn't sample it (sampling is biased to high-load/slow queries)
- DBM is not correctly configured or enabled for that database
- For SQL Server: `lookback_window` for query metrics may be too short

**What to Check:**

1. Confirm DBM requirements: https://docs.datadoghq.com/database_monitoring/connect_dbm_and_apm/?tab=go#before-you-begin
2. Verify Agent version and DBM configuration for that database
3. For SQL Server, consider increasing `lookback_window` (especially for infrequent queries)
4. Check Agent flare for errors in query sample / activity collection

**Escalation:** DBM / #support-database-monitoring

---

### Bucket B: DBM has Sample/Plan but No Comments and/or No DB Span

**Scenario:**
- Question 1 = "Yes" (sample/plan exists)
- Question 2 = "No" (no comments) AND/OR
- Question 3 = "No DB span" (no DB span in APM)

**Interpretation:**
The DB query is observed by DBM, but:
- The APM tracer did not create DB spans, OR
- DB spans exist but did not inject comments into the SQL

**What to Check:**

1. Confirm APM tracing & DBM–APM requirements:
   - Tracing collection: https://docs.datadoghq.com/tracing/trace_collection/
   - DBM + APM: https://docs.datadoghq.com/database_monitoring/connect_dbm_and_apm/?tab=go#before-you-begin

2. From tracer debug logs, confirm:
   - Are DB spans created at all (spans with `.query` etc.)?
   - Are they being sent to the Agent?

3. Check hidden span metadata:
   - If no `dbm_trace_injected` → tracer is not injecting DBM comments (propagation not configured, unsupported library, wrong version, etc.)

4. Obtain from the user:
   - Code snippet showing how the DB call is made
   - Tracer configuration / environment variables (`DD_DBM_PROPAGATION_MODE`, etc.)
   - Tracer debug logs around the time the query runs

**Escalation:**
- APM / #support-apm for span creation/propagation issues
- DBM if comments appear to be stripped by DB or Agent

---

### Bucket C: Sample Has Comments, but Trace is Missing or Not Correlated

**Scenario:**
- Question 1 = "Yes" (sample/plan exists)
- Question 2 = "Yes" (comments exist)
- Question 3 = "Trace missing / not found / not linked"

**Interpretation:**
At some point, a DB span existed and injected comments (DBM captured them), but:
- The trace/span was not retained (sampling, retention, ingestion), OR
- The query text/signatures do not match, so correlation cannot be resolved

**What to Check:**

1. Confirm APM ingestion and retention settings (intelligent retention, custom filters, etc.)
2. Confirm DBM Agent configuration:
   - Ensure `collect_comments` is not disabled in `obfuscator_options` for that DB
3. Check normalized vs exact statement matching:
   - Compare APM and DBM query text
   - Compare `_dd.query_signature` and `@trace.span.resource_hash` if present

**Escalation:**
- APM / #support-apm for ingestion/retention/sampling issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddalexvea) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
