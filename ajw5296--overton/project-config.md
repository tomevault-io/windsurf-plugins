---
trigger: always_on
description: PSU Research Impact Dashboard — combines data from Penn State RMD (Researcher Metadata Database), OpenAlex, and Overton to track how Penn State research is cited in policy documents. Streamlit dashboard deployed on ECS Fargate, backed by a 6-stage data pipeline orchestrated by Step Functions with Lambda fan-out, writing to RDS PostgreSQL + S3.
---

# CLAUDE.md

## Project Overview

PSU Research Impact Dashboard — combines data from Penn State RMD (Researcher Metadata Database), OpenAlex, and Overton to track how Penn State research is cited in policy documents. Streamlit dashboard deployed on ECS Fargate, backed by a 6-stage data pipeline orchestrated by Step Functions with Lambda fan-out, writing to RDS PostgreSQL + S3.

## Architecture

- **Database**: RDS PostgreSQL (db.t3.micro) with JSONB columns — `overton-postgres.c86jss96xcpx.us-east-1.rds.amazonaws.com`
- **Storage**: S3 bucket `overton-datalake-700032885189` for raw API response archival, PDF documents, and Lambda deployment packages
- **Dashboard**: Streamlit on ECS Fargate (existing `overton-dashboard` stack)
- **Pipeline**: 6-stage ETL via Lambda functions, with fan-out parallelism for stages 3-5
- **Orchestration**: Step Functions with Map states for fan-out + EventBridge for weekly scheduling
- **Region**: us-east-1
- **VPC**: vpc-b653f0cb (default VPC)
  - Public subnets: EC2 dev instance, NAT Gateway, ECS tasks
  - Private subnets: `subnet-05e2b3532a74fd147` (us-east-1a), `subnet-024916f3b1d8b27a7` (us-east-1b) — Lambda functions route through NAT Gateway for internet access while maintaining VPC access to RDS

## Pipeline Stages

```
Stage 1: RMD Cohort         → researchers table  (cohort + grants/profile + RMD DOI list)
Stage 2: OpenAlex Enrichment → researchers (update: works_dois, metrics, flags)
Stage 3: Overton DOI-set    → articles + article_citations + policy_documents stubs
Stage 4: Overton Documents  → policy_documents (enrich stubs)
Stage 5: PDF Download       → S3 (with pre-check to skip already-downloaded)
Stage 6: Export             → flat JSON files for dashboard fallback
```

**Cohort = ~1,011 researchers** = 752 ORCID-mapped (real ORCID PK) + 259 unmapped (`wa:<webaccess_id>` synthetic PK). Built by `_scan_rmd_publications()` walking all 17 HHD orgs.

**Stage 2 author resolution**: real ORCID → `/authors/orcid:X`; synthetic ID → `/authors?search=<name>&filter=ror:PSU` + `pipeline.name_match` matcher. Auto-accept only when PSU `last_known_institution` AND name matches at top tier; everything else → review queue.

**Disambiguation guard** (Stage 2): if `oa_works/rmd_works > 5×` AND `overlap < 10%`, the OpenAlex author record is conflated; flag suspect, drop OA DOIs, write to review file.

**Stage 3 DOI-set flow**: per researcher, union RMD + OpenAlex DOIs → POST `/generate_id_set.php` (newline-separated DOIs, must include `Content-Type: application/x-www-form-urlencoded` header) → GET `/articles.php?dois=<set_id>`. Article responses contain nested `cited_by_documents` for the policy graph.

**Articles table** holds every PSU work, not just policy-cited ones. Stage 3 stubs the OA-only DOIs; only the Overton-tracked subset gets full metadata.

**Run locally**:
```bash
export AWS_DEFAULT_REGION=us-east-1
export DATABASE_SECRET_ARN="arn:aws:secretsmanager:us-east-1:700032885189:secret:overton/rds-credentials-M85fjC"
export S3_BUCKET_NAME="overton-datalake-700032885189"
export OPENALEX_EMAIL="overton-pipeline@psu.edu"
export OVERTON_API_KEY="<from secretsmanager: overton/api-keys/overton>"
export PSU_RESEARCH_API_KEY="<from secretsmanager: overton/api-keys/rmd>"

python -m pipeline.run --rebuild-map               # ~20 min, builds ORCID/DOIs/names maps in S3
python -m pipeline.run --all                       # full pipeline end-to-end
python -m pipeline.run --stage <stage>             # one stage only
python -m pipeline.run --all --max-researchers 50  # smoke test
```

**Cut-over runbook** (when schema changes need fresh tables):
```bash
python -m scripts.drop_pipeline_tables --confirm   # drops 4 tables, keeps pipeline_metadata + S3 PDFs
python -m pipeline.run --rebuild-map
python -m pipeline.run --all
```

## Database Tables

- `researchers` — PK: `orcid` (real ORCID or `wa:<webaccess_id>` synthetic). JSONB `data` with:
  - `openalex.works_count`, `cited_by_count`, `h_index`, `topics`, `affiliations`, **`works_dois`** (full DOI list)
  - `rmd.profile`, `grants`, `presentations_count`, `etds_count`, `org_memberships`, **`dois`** (RMD-known DOI list), **`name`** (real person name from publications scan)
  - `flags.lookup_method` ('orcid' or 'name_search'), `flags.oa_disambiguation_suspect` (bool)
  - `discovered_orcid` (when name search resolved an ORCID we didn't know)
- `articles` — PK: `doi`. Columns: `data` (JSONB), `policy_citation_count` (int), `last_policy_cited_at` (timestamp), `source_set` (JSONB array of `{rmd, openalex, overton}`). Holds every PSU work; non-policy-cited rows are stubs with just the DOI.
- `policy_documents` — PK: `policy_document_id`. Columns: `data` (JSONB w/ Overton metadata), `pdf_url`, `s3_pdf_key`, `download_status` (`pending`|`downloading`|`downloaded`|`skipped_too_large`|`failed`), `dont_show_pdf`. Stub rows with just `{policy_document_id}` get enriched by Stage 4.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajw5296/overton](https://github.com/ajw5296/overton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
