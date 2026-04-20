---
trigger: always_on
description: **ALL infrastructure changes MUST be implemented via Terraform.** Never make infrastructure changes directly via AWS CLI, API, or console and call it done — the change must be codified in `.tf` files so it is reproducible, reviewable, and version-controlled.
---

# Erudite-s Project Defaults

## Rule #1: Terraform-First and Branch-PR Workflow (ABSOLUTE - STRICT - NEVER VIOLATE)

**ALL infrastructure changes MUST be implemented via Terraform.** Never make infrastructure changes directly via AWS CLI, API, or console and call it done — the change must be codified in `.tf` files so it is reproducible, reviewable, and version-controlled.

**ALL code changes MUST go through a branch and PR.** Never push directly to `main`. The workflow is always: branch → commit → push → open PR → review/CI passes → merge.

### NEVER DO
- **NEVER make infrastructure changes only via CLI/API** without updating Terraform to match
- **NEVER push directly to main** — always use a branch and PR
- **NEVER consider a manual change "done"** — it must be in Terraform before it's done
- **NEVER skip the PR process** even for "small" changes

### ALWAYS DO
- **Implement infrastructure changes in Terraform first**, then apply
- **If an emergency change was made via CLI/API**, immediately create a PR to import/codify it in Terraform
- **Use `terraform import`** to bring manually-created resources under Terraform management
- **Create a branch, push, and open a PR** for every change, no exceptions

## Fully Automated Build & Deploy Pipeline (ABSOLUTE - STRICT - NEVER VIOLATE)

**ALL build, push, and deploy operations MUST be 100% automated via Terraform and GitHub Actions pipelines. Zero manual intervention, zero SSH, zero manual container builds.**

### The ONLY Permitted Workflow
```
Code change → git commit → git push → Open PR → Merge PR → GitHub Actions pipeline:
  1. Build Docker image(s)
  2. Push to ECR
  3. Sync configs to S3
  4. Trigger EC2 service restart (via SSM Run Command, not SSH)
  5. Verify health checks pass
```

### NEVER DO (applies to ALL erudite-s repos and child folders)
- **NEVER run `docker build` locally** to build deployment images — the pipeline builds them
- **NEVER run `docker push` locally** to push images to ECR — the pipeline pushes them
- **NEVER SSH into EC2** to run `docker compose up`, restart services, set env vars, or sync configs
- **NEVER run `aws ecr get-login-password` locally** as a deployment step
- **NEVER run `aws s3 sync` locally** to deploy configs — the pipeline syncs them
- **NEVER manually export secrets/env vars on a server** to make Docker Compose work
- **NEVER use `sudo -E docker compose up -d`** over SSH — this is ClickOps
- **NEVER run `terraform init/plan/apply` locally** with AWS profiles — always trigger via GHA deploy workflow
- **NEVER use local AWS_PROFILE to test apps** designed for EC2 IAM instance roles — test on the actual EC2 via SSM

### ALWAYS DO
- **Fix the pipeline** if it doesn't exist or is broken — creating automation is the job, not working around it
- **Use Terraform** for infrastructure changes (EC2, RDS, S3, IAM, security groups, etc.)
- **Use GitHub Actions** for build, push, deploy, and verification steps
- **Use SSM Run Command or user-data.sh** for server-side operations — never interactive SSH
- **Use AWS Secrets Manager** for runtime secrets — never hardcode or manually export them
- **Verify deployments via health check endpoints** in the pipeline — never by SSHing in to check

### Formpak AI PoC (`erudite-s/formpak-ai-poc`) — EC2 GPU Architecture
- **Instance type: g6.xlarge** (NVIDIA L4, 23GB VRAM) — single EC2, not SageMaker
- **Architecture: 4 containers** on one instance — inference (BGE-M3 + 2 rerankers + TATR), postgres (PG18 + pgvector + AGE + db2_fdw), db2, app (FastAPI)
- **Inference API**: port 8080, endpoints: `/health`, `/models`, `/predict`
- **App API**: port 8100, 21 endpoints including `/ai/query`, `/ai/chat`, `/ai/agent`
- **LLM**: Anthropic API direct (ChatAnthropic via `create_llm()`), no Bedrock
- **Single AWS account: 572444560619 / eu-west-2** — Terraform in `formpak-ai-poc/terraform/`, `terraform init -backend-config=backend.hcl`
- **Platform healthcheck**: `scripts/platform-healthcheck.sh` — 36-check E2E test harness. Run via SSM: `aws ssm send-command --parameters commands=["bash /opt/formpak-ai/platform-healthcheck.sh"]`. Supports `--full` (default) and `--quick` modes.
- **DevOps refactoring plan**: `.planning/devops-refactoring/PLAN.md` — 5 phases, Phases B-E remaining

### GitHub Org Secrets & Variables (Erudite-s)
Shared credentials are stored at **org level** (visibility: ALL), not duplicated per-repo.
**RULE: Tokens MUST be stored as BOTH a secret AND a variable.** Workflows use `secrets.*`, MCP/tools use `vars.*`. Missing either causes silent failures.

- **SonarQube**: `SONAR_TOKEN`, `SONARQUBE_MCP_TOKEN` (both as secret + variable), `SONAR_HOST_URL` (variable only — not sensitive)
- **Component Test DBs**: `COMPONENT_TEST_{DB2,MSSQL,POSTGRES}_{URL,USER,PASSWORD}` (9 secrets)

### Terraform Sensitive Variables (Erudite-s/infrastructure repo)
Terraform variables are provided via **repo-level GitHub Secrets** on `Erudite-s/infrastructure` (not org-level). The GHA deploy workflow passes them as `TF_VAR_*` env vars.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Erudite-s) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
