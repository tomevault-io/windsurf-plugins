---
trigger: always_on
description: Orientation map for AI agents. Optimized for fast productivity, not exhaustiveness. Verify against cited files before making claims.
---

# AGENTS.md — One Observability Demo

Orientation map for AI agents. Optimized for fast productivity, not exhaustiveness. Verify against cited files before making claims.

## 1. Purpose

Sample "pet adoption" application powering the [AWS One Observability Workshop](https://observability.workshop.aws/). It is a polyglot microservices app (6 services in 6 languages) plus supporting Lambdas, deployed entirely by an AWS CDK **self-mutating pipeline**. The point is to demonstrate AWS observability (CloudWatch, Application Signals, X-Ray, ADOT/OpenTelemetry, OpenSearch). Docs are published at https://aws-samples.github.io/one-observability-demo/.

## 2. TL;DR for agents

- Everything lives under `src/`. Two halves: **infrastructure** (`src/cdk/`, TypeScript CDK) and **application code** (`src/applications/`, polyglot).
- The CDK app is the control plane. Two entry points: `src/cdk/bin/workshop.ts` (production pipeline) and `src/cdk/bin/local.ts` (direct stack deploy for local dev). `cdk.json` runs `bin/workshop.js`.
- Deployment model: a CDK **CodePipeline** deploys 5 stages — Core, Containers, Storage, Compute, Microservices (see `src/cdk/lib/pipeline.ts` and `lib/stages/`).
- 6 microservices in `src/applications/microservices/`: Go, Rust, Java, Python, .NET, Python-Strands-agent. Each has its own Dockerfile and build toolchain.
- Config is driven by env vars resolved in `src/cdk/bin/environment.ts` + `constants.ts`; presets in `src/presets/*.env`; local dev uses `src/cdk/.env` (copy from `.env.sample`).
- CI: GitHub Actions in `.github/workflows/` (build-test, tests, security-scan/ASH, pre-commit, docs, release). Pre-commit hooks are heavy (secrets, lint, jest, ASH) — see `.pre-commit-config.yaml`.
- Commits must follow **Conventional Commits** (commitizen enforced on commit-msg).
- `archive/` is a scratch/temp folder slated for removal — do not treat as authoritative. `buildspec.yml` references an obsolete `PetAdoptions/cdk/pet_stack/` path (stale; real build is CDK pipeline).
- Only touch code inside its component directory; the repo root `package.json` is just tooling (eslint/typedoc), not the app.

## 3. Repository map

| Path | Purpose | Entry points / key files |
|------|---------|--------------------------|
| `src/cdk/` | CDK app (TypeScript) — all infra | `bin/workshop.ts`, `bin/local.ts`, `bin/environment.ts`, `cdk.json`, `index.ts` |
| `src/cdk/lib/pipeline.ts` | Self-mutating CDK CodePipeline definition | `CDKPipeline` class |
| `src/cdk/lib/stages/` | The 5 deployment stages (Stack + Stage per file) | `core.ts`, `containers.ts`, `storage.ts`, `compute.ts`, `applications.ts` |
| `src/cdk/lib/constructs/` | Reusable L3 constructs (vpc, eks, ecs, database, waf, opensearch, lambda, canary…) | `index.ts` re-exports |
| `src/cdk/lib/microservices/` | Per-service CDK wiring + EKS `manifests/` | `petsite.ts`, `pet-search.ts`, `petfood.ts`, … |
| `src/cdk/lib/serverless/` | Lambda + canary CDK definitions | `functions/`, `canaries/` |
| `src/cdk/scripts/` | Ops scripts (deploy-check, seed, redeploy, cleanup, validate-account) | `deploy-check.sh`, `seed-dynamodb.sh`, `redeploy-app.sh` |
| `src/applications/microservices/` | The 6 runtime services (polyglot) | one dir per service (see §7) |
| `src/applications/lambda/` | 9 Lambda functions (Node/Python) | e.g. `petstatusupdater-node`, `rds-seeder-python` |
| `src/applications/canaries/` | CloudWatch Synthetics canaries (Node) | `petsite-canary`, `housekeeping` |
| `src/templates/` | CloudFormation bootstrap template for the workshop | `codebuild-deployment-template.yaml` |
| `src/presets/` | Env presets consumed by CDK config | `default.env`, `hardened.env`, `workshop.env` |
| `src/scripts/` | Pet image generation (Python) | `realistic_*.py` |
| `docs-site/` | MkDocs site (published docs) | `mkdocs.yml`, `docs/` |
| `static/images/` | Pet image assets seeded to S3 | bunnies/kittens/puppies |
| `generated-diagrams/` | Architecture diagram images | — |
| `archive/` | Temp scratch (gitops, grafana, keycloak) — slated for removal | `README.md` says so |
| `.github/workflows/` | CI | see §5 |

## 4. Tech stack

- **Infra:** AWS CDK v2 (`aws-cdk-lib` 2.241, TypeScript ~5.9), cdk-nag, CodePipeline/CodeBuild. Node 22.
- **Microservices:** Go (`payforadoption-go`), Rust/Axum (`petfood-rs`), Java/Spring Boot + Gradle (`petsearch-java`), Python/FastAPI (`petlistadoptions-py`), .NET 8 (`petsite-net`), Python/Strands agent on Bedrock AgentCore (`petfoodagent-strands-py`).
- **Runtimes/platforms:** ECS Fargate (most services), EKS (petsite), Bedrock AgentCore (agent), Lambda (Node 22 / Python 3.13).
- **Data:** Aurora PostgreSQL, DynamoDB, S3, SQS, EventBridge, OpenSearch Serverless.
- **Observability:** CloudWatch, Application Signals, X-Ray, ADOT/OpenTelemetry, Prometheus, CloudWatch Synthetics, RUM.
- **Docs:** MkDocs (Material) + TypeDoc for CDK API reference.

## 5. Build / test / run / deploy

**CDK infra** (from `src/cdk/`, per `src/cdk/package.json` + `.github/workflows/tests.yml`):
- Build: `npm run build` (tsc). Type check: `npx tsc --noEmit`.
- Test: `npm run test` (jest). List stacks: `npx cdk ls`. Synth: `npx cdk synth`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/one-observability-demo](https://github.com/aws-samples/one-observability-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
