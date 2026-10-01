---
trigger: always_on
description: Single entry point and source of truth for coding agents (and humans) working in this repository.
---

# AGENTS.md — Well-Architected IaC Analyzer

Single entry point and source of truth for coding agents (and humans) working in this repository.
It describes what the project is, how the pieces fit together, where everything lives, how to build
and verify changes, and the conventions and traps you must respect. `CLAUDE.md` contains only
`@AGENTS.md` (an import of this file); `README.md` is the end-user deployment guide.

---

## 1. What this project is

**Well-Architected IaC Analyzer** is an AWS sample (MIT-0, explicitly *non-production*) that uses
Amazon Bedrock to review infrastructure artifacts against AWS Well-Architected best practices:

- **Inputs:** a single IaC file (CloudFormation YAML/JSON, Terraform, CDK source in TS/PY/GO/JAVA/C#),
  a multi-file IaC project (multiple files or a `.zip`), an architecture diagram (PNG/JPEG), or up to
  five architecture PDFs. An optional *supporting document* (PDF/TXT/PNG/JPEG ≤ 4.5 MB) adds context.
- **Analysis:** for every question of the selected **lens** (the WA Framework or one of 15 AWS official
  lenses, plus customer **custom lenses**) and selected pillars, the backend retrieves guidance from a
  Bedrock Knowledge Base (RAG) and asks a Claude model to mark each best practice
  *relevant / applied / not applied*, with reasons, recommendations and a **prioritization**
  (Criticality, Complexity, Priority → Eisenhower matrix).
- **Outputs:** an interactive results table, a Priorities matrix, CSV export, "Get more details"
  Markdown deep-dives, IaC template generation from diagrams, a chat assistant grounded in the
  results, and push-back of answers/milestones/reports into the **AWS Well-Architected Tool**.
- **Persistence:** every upload is a *work item* (DynamoDB + S3) keyed per user and per lens, so
  results survive dropped connections and can be reloaded from the side navigation.

It ships as **three deliverables that must stay consistent**:

| # | Deliverable | Location |
|---|-------------|----------|
| 1 | NestJS backend + React/Cloudscape frontend, containerized, on ECS Fargate | `ecs_fargate_app/backend`, `ecs_fargate_app/frontend`, `ecs_fargate_app/finch` |
| 2 | Python CDK stack that builds the whole environment (network, ECS, ALB+auth, Bedrock KB, DynamoDB, S3, Lambdas) | `app.py`, `ecs_fargate_app/wa_genai_stack.py` |
| 3 | CloudFormation "deployment stack": a throw-away EC2 that clones this repo from GitHub, rewrites `config.ini`, runs the CDK deploy, then self-deletes | `cfn_deployment_stacks/iac-analyzer-deployment-stack.yaml` |

Upstream: `https://github.com/aws-samples/well-architected-iac-analyzer` (branch `main`).
Dependabot PRs are the normal dependency-update path — versions are pinned everywhere.

---

## 2. Tech stack (pinned versions as of this file)

| Layer | Technology |
|-------|-----------|
| Infra-as-code | Python 3.11+ CDK v2 (`aws-cdk-lib==2.253.0`, `cdklabs.generative_ai_cdk_constructs==0.1.312`, `cdk-nag`), CDK CLI ≥ 2.x, Docker or Finch for image assets |
| Backend | Node ≥ 20.19 / 22.12 at runtime (Nest 12 packages are ESM-only and load via `require(esm)`; images use `node:alpine3.23`, the CFN deployer installs Node 24), NestJS 12 (`@nestjs/common`, `core`, `platform-express`, `platform-socket.io`, `websockets` 12.0.3, `@nestjs/config` 12.0.0, `@nestjs/throttler` 6.7.0; `@nestjs/cli` 11.0.21 / `schematics` 11.1.0 kept on 11 because schematics 12 requires TypeScript ≥ 6), `socket.io` 4.8.3, AWS SDK v3 `3.1000.0` (bedrock-runtime, bedrock-agent-runtime, dynamodb, s3, wellarchitected, lib-storage), `adm-zip` 0.6.0, `class-validator` 0.14.3, TypeScript 5.9.3 (CommonJS, ES2021, **lenient**: `strictNullChecks: false`, `noImplicitAny: false`) |
| Frontend | React 19.2.3, Vite 8.0.16, TypeScript 5.9.3 (**strict**, `noUnusedLocals`/`noUnusedParameters` on), Cloudscape (`components` 3.0.1174, `chat-components` 1.0.89, `code-view` 3.0.94, `design-tokens` 3.0.68, `global-styles` 1.0.49, `collection-hooks`), `axios` 1.18.0, `socket.io-client` 4.8.3, `react-markdown` 10.1.0, `react-syntax-highlighter` 16.1.0, `react-rnd` 10.5.2 |
| Containers | `node:alpine3.23` (backend, multi-stage), `nginx:stable-alpine-slim` (frontend, serves SPA + reverse proxy), both run as non-root |
| Lambdas | Python (latest runtime via `determine_latest_python_runtime`), `boto3==1.37.2` (migration, cleanup), `requests` (KB synchronizer) |
| AI | Amazon Bedrock Converse API (Claude family), Bedrock Knowledge Base `RetrieveAndGenerate` with Titan Embed Text v2 (1024-dim), vector store = **S3 Vectors** (default) or OpenSearch Serverless |
| Data | DynamoDB (`AnalysisMetadataTable`, `LensMetadataTable`), S3 (analysis storage, WA docs/KB source, access logs), SSM Parameter Store (custom lenses) |
| Edge / identity | ALB (HTTPS, TLS 1.3 policy) with Cognito or OIDC `authenticate-*` listener actions; no app-level login |

No test suite and no ESLint config exist in the repo (`npm run lint` fails in both packages; `vitest`
is a devDependency without tests/config). **Type-checking via `npm run build` is the only automated
verification** for TS; `cdk synth` (with `cdk_nag`) is the gate for the CDK.

---

## 3. Repository layout (every tracked file)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/well-architected-iac-analyzer](https://github.com/aws-samples/well-architected-iac-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
