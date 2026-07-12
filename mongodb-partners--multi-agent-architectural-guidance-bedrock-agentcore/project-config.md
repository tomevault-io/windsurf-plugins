---
trigger: always_on
description: This file is for **human and AI contributors** who edit the repository in Cursor, Copilot, or similar tools. It is **not** a Strands/Bedrock agent definition.
---

# Guidance for AI coding agents

This file is for **human and AI contributors** who edit the repository in Cursor, Copilot, or similar tools. It is **not** a Strands/Bedrock agent definition.

**Runtime agent personas** live under [`config/agents/`](config/agents/) as `.agent.md` files. **Domain knowledge** lives under [`config/skills/`](config/skills/) as `SKILL.md` trees.

---

## What this project is

A **configuration-driven multi-agent reference** on **AWS Bedrock** (via [Strands Agents SDK](https://github.com/strands-agents/sdk-typescript)) and **MongoDB Atlas**. Add specialists by editing markdown config, not by forking business logic for every customer.

**Request path (production):** Streamlit UI → Hono API → **in-API classifier** (`agent-classifier.ts`) → specialist **AgentCore Runtime** (single hop). Mongo tools go through a dedicated **MongoDB MCP AgentCore Runtime**. `USE_ORCHESTRATOR_RUNTIME=1` enables a two-hop rollback through the orchestrator runtime.

**Five AgentCore Runtimes:** orchestrator + 3 specialists (`order-management`, `product-recommendation`, `troubleshooting`) + MongoDB MCP.

**Connectivity modes** (mutually exclusive per account+region): `NETWORK_MODE=privatelink` (default) or `NETWORK_MODE=peering`. Switching requires destroy + redeploy.

**Getting started:** [`docs/README.md`](docs/README.md) — doc map, first-day checklist, reading orders.

**Debugging:** [`docs/status/debugging.md`](docs/status/debugging.md) — EC2 access, trace-driven debug, common failures, validation scripts, **persistent pitfalls**. When a non-obvious regression recurs more than twice (or is a severe guardrail like hung CI / infinite Strands loop), add an entry to **Known persistent pitfalls** in the same PR as the fix.

**Reference appendix:** [`docs/reference/`](docs/reference/) — env vars, tools, Terraform modules, SSM parameters, data model, smoke tests, deploy scripts.

---

## Getting started (new contributor)

> **Just cloned?** Read [`README.md`](README.md) first for the full walkthrough, then return here when you change code.

### Step 1 — Install tools

Bun, Python 3.10+, AWS CLI v2, Terraform ≥ 1.6, Docker (EC2 deploys), `zip`/`curl`. Install commands: [README § Prerequisites](README.md#prerequisites).

### Step 2 — Configure `.env`

```bash
git clone <repo-url>
cd mongodb-aws-bedrock-multi-agent-framework

cp .env.sample .env    # never commit this file
```

Edit `.env` before any deploy. Minimum fields (see [`.env.sample`](.env.sample) for the full list):

| Section | What to set |
|---|---|
| AWS auth | `AUTH_MODE`, then either IAM keys or STS/`AWS_PROFILE` (see [`deploy/iam/README.md`](deploy/iam/README.md)) |
| Region + identity | `AWS_REGION`, `ENVIRONMENT`, `PROJECT_NAME`, `SHARED_VPC_NAME` |
| Atlas | `MONGODB_ATLAS_PUBLIC_KEY`, `MONGODB_ATLAS_PRIVATE_KEY`, `TF_VAR_mongodb_atlas_org_id`, `TF_VAR_mongodb_atlas_project_id`, `TF_VAR_atlas_db_password` |
| Embeddings | `EMBEDDINGS_PROVIDER` is **mandatory** in every environment (deployed and local). [`.env.sample`](.env.sample) defaults to `titan` (simplest first-deploy path — no Voyage Marketplace required). Switch to `voyage` for Voyage multimodal embeddings. Strict — no implicit default, no cross-provider fallback at runtime. Both `assertEmbeddingsProvider()` (API boot) and `deploy-project.sh` refuse to run if it's unset / unrecognised. |

Then:

```bash
source .env
aws sts get-caller-identity    # must succeed before deploy
```

### Step 3 — Install app dependencies

```bash
export PATH="$HOME/.bun/bin:$PATH"
cd api && bun install && cd ..
cd ui && pip install -r requirements.txt && cd ..
```

### Step 4 — Deploy and run

The API **refuses to boot** without `AUTH_JWKS_URI`, `AUTH_ISSUER`, and `AGENTCORE_ORCHESTRATOR_ARN` ([boot guards](#api-boot-guards)). A fresh clone cannot run `bun run dev` until a deploy writes those into `.env.live`.

#### Recommended first-time path (runnable chat stack)

Full EC2 deploy (~30–45 min). Provisions Cognito, AgentCore runtimes, EC2, Atlas, KB — everything needed to chat.

```bash
source .env
./deploy/deploy-full-with-privatelink.sh --auto-approve
# Full post-deploy smoke runs in deploy-project.sh Phase 11 (use --skip-smoke to skip).
# Re-run manually: source .env && python3 e2e-smoke/post-deploy-smoke.py
```

When the script finishes, open the **UI URL** it prints (Streamlit on EC2, port 8501). Sign in with the Cognito test user from the deploy output.

Alternative connectivity: `./deploy/deploy-full-with-vpc-peering.sh --auto-approve` (mutually exclusive with PrivateLink).

Bring your own MongoDB Atlas cluster (demo only): `./deploy/deploy-full-public.sh --auto-approve` — points at your *own* existing Atlas cluster over the public internet (`NETWORK_MODE=public` + `ATLAS_CLUSTER_SOURCE=byo`, requires `MONGODB_BYO_URI` and a `0.0.0.0/0` Atlas access list). Use `EMBEDDINGS_PROVIDER=titan` for this setup — no SageMaker endpoint to provision.

Deploy scripts run centralized preflight checks before mutating AWS resources.

#### Run API + UI on your laptop (after a full EC2 deploy)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb-partners/Multi-Agent-Architectural-Guidance-Bedrock-AgentCore](https://github.com/mongodb-partners/Multi-Agent-Architectural-Guidance-Bedrock-AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
