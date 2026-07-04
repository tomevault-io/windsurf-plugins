---
trigger: always_on
description: Context for coding agents (Claude Code, Cursor, etc.) working on this codebase. End users read [README.md](./README.md); this file covers what an agent needs that isn't obvious from the code.
---

# Agent guide — Public Comment Analyzer

Context for coding agents (Claude Code, Cursor, etc.) working on this codebase. End users read [README.md](./README.md); this file covers what an agent needs that isn't obvious from the code.

## What this is

A serverless AWS application that uses AWS Bedrock (Claude) to analyze public comments from CSV/XLSX files. Users upload a file, define analysis columns, get back the original data plus AI-generated columns and an aggregate summary.

Stack: Angular 21 frontend on CloudFront/S3, Python 3.12 Lambdas behind API Gateway, S3 for data, DynamoDB for job state, Bedrock for inference, AWS Secrets Manager for the access password.

## Critical conventions

### AWS profile
Every AWS CLI / CDK invocation must use `--profile $AWS_PROFILE`. The profile name comes from `.env` (copy from `.env.example`). Don't hardcode profile names in scripts or docs — `ncdit` is NC-specific and only valid on the maintainer's machine.

### Python environment
- **Python 3.12** is required (Lambda runtime is `python3.12` on Amazon Linux 2023, glibc 2.34).
- Use the project `.venv` if it already exists. Don't run bare `python` for installs — always use `.venv/bin/python` or `source .venv/bin/activate` first.
- Never bump the Lambda runtime down to 3.11 or earlier — modern wheels of `bcrypt` (and others) require glibc 2.28+ and won't load on AL2.

### No hardcoded account / domain identifiers
This repo is open source. Never commit:
- AWS account IDs
- CloudFront distribution IDs / subdomains
- ACM certificate ARNs
- Production domain names
- Password literals or hashes

NC-specific values live in GitHub Actions secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `DOMAIN_NAME`, `CERTIFICATE_ARN`, `ALLOWED_ORIGIN`) and in the operator's local `.env` / `local-env.json` (gitignored). The CDK stack reads them via `--context` flags; missing context falls back to safe defaults (no custom domain, CORS `*` for dev).

If you find a hardcoded NC value while editing code, parameterize it.

## Architecture

### Lambdas
Located under `backend/<name>/handler.py`:

1. **upload_handler** — file validation (size, type, magic bytes, filename sanitization), stores uploads in S3, creates a DynamoDB job record.
2. **row_processor** — fans out per-row processing across a 500-worker `ThreadPoolExecutor`, calls Claude Haiku via Bedrock, writes results back to S3, then async-invokes the aggregate analyzer.
3. **aggregate_analyzer** — calls Claude Opus for a holistic summary, writes results to S3, marks the job complete in DynamoDB.
4. **dashboard_generator** — builds Chart.js-compatible JSON from the per-row results.
5. **auth_handler** — validates the shared password against the bcrypt hash in Secrets Manager (or `LOCAL_PASSWORD_HASH` for local dev).
6. **status_handler** — DynamoDB job-status polling.

### Shared layer
`backend/shared/` is built into a Lambda Layer that all functions consume:
- `auth.py` — bcrypt password validation, fail-closed if no secret is configured.
- `file_parser.py` / `file_writer.py` — CSV/XLSX I/O. `file_writer` neutralizes formula-injection (cells starting with `=`, `+`, `-`, `@`, tab, CR get prefixed with `'`).
- `dynamodb_client.py` — typed wrapper around the jobs table.

When you change anything in `backend/shared/`, the next deploy rebuilds the layer and every consuming Lambda picks it up automatically — no per-function publish step.

### Data flow
```
Upload → S3 → DynamoDB (job created) → RowProcessor (concurrent)
       → S3 (per-row results) → AggregateAnalyzer
       → DynamoDB (complete) → frontend downloads results
```

### Concurrency
- 500 ThreadPoolExecutor workers per RowProcessor invocation.
- Lambda reserved concurrency: 500.
- Bedrock account quota: 1,000 req/min (we run at ~50% to leave headroom for retries).
- DynamoDB writes are batched: progress updates every 50 rows, not every row.

If you tune concurrency, change all three values (worker pool, reserved concurrency, the env var that gates the worker count) — they need to stay in sync.

## Security

### Auth
- Stored format: **bcrypt** hash (e.g. `$2b$12$...`) in Secrets Manager under `PublicCommentAnalyzer-AccessPassword-<env>`.
- Verified with `bcrypt.checkpw` (constant-time). Never use `==` or SHA-256 — both have appeared in this repo's history and were both wrong.
- `validate_access_key` fails closed: if neither `ACCESS_PASSWORD_SECRET_NAME` nor `LOCAL_PASSWORD_HASH` is configured, every request is rejected.
- Frontend holds the access key **in memory only** on the root-scoped `AuthService` singleton (`auth.service.ts`) — never in `sessionStorage`/`localStorage`. This keeps the credential out of any JS-readable web storage (the prior `sessionStorage` approach tripped CodeQL `js/clear-text-storage-of-sensitive-data` / Checkmarx CWE-922). The `authInterceptor` reads the key via `AuthService.getAccessKey()`, not from storage. Trade-off: a full page reload clears it and re-shows the access gate — acceptable since job state isn't persisted across reloads either. Do not regress this back to web storage.

### Prompt injection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NC-DIT-Open-Source/Public-Comment-Analyzer](https://github.com/NC-DIT-Open-Source/Public-Comment-Analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
