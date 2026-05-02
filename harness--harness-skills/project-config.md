---
trigger: always_on
description: Use when a user wants to deploy a new service to Harness. Follow this exact order:
---

# Harness Skills

This repository contains Claude Code skills for working with Harness.io CI/CD platform. All skills use the Harness MCP v2 server's consolidated tool interface.

## MCP v2 Server

All skills use the [Harness MCP v2](https://github.com/thisrohangupta/harness-mcp-v2) server which provides 10 generic tools operating across 119+ resource types:

| Tool | Purpose |
|------|---------|
| `harness_list` | List resources with filters and pagination |
| `harness_get` | Get a single resource by ID |
| `harness_create` | Create a resource (requires confirmation) |
| `harness_update` | Update a resource (requires confirmation) |
| `harness_delete` | Delete a resource (requires confirmation) |
| `harness_execute` | Run, retry, interrupt, sync, toggle, approve, reject, test_connection |
| `harness_search` | Cross-resource keyword search |
| `harness_describe` | Local metadata/schema lookup (no API call) |
| `harness_diagnose` | Pipeline failure analysis |
| `harness_status` | Project health overview |

Tools accept a `resource_type` parameter (e.g., `pipeline`, `secret`, `template`) to target specific Harness resources. Tools also support Harness UI URL auto-extraction for `org_id`, `project_id`, `resource_type`, and `resource_id`.

## Skill Directory

Skills live in `skills/<skill-name>/SKILL.md`. Each skill folder may contain `references/`, `scripts/`, and `assets/` subdirectories.

### Pipeline & Execution

| Skill | Description |
|-------|-------------|
| `/create-pipeline` | Generate v0 pipeline YAML (CI, CD, combined, approvals) |
| `/create-pipeline-v1` | Generate v1 simplified pipeline YAML |
| `/create-trigger` | Create webhook, scheduled, and artifact triggers |
| `/create-template` | Create reusable step, stage, pipeline, and step group templates |
| `/run-pipeline` | Execute and monitor pipeline runs |
| `/debug-pipeline` | Diagnose pipeline execution failures |
| `/migrate-pipeline` | Convert v0 pipelines to v1 format |

### Infrastructure & Resources

| Skill | Description |
|-------|-------------|
| `/create-service` | Define services (Kubernetes, Helm, ECS) with artifact sources |
| `/create-environment` | Create environments (PreProduction, Production) with overrides |
| `/create-infrastructure` | Define infrastructure (K8s, ECS, Serverless) |
| `/create-connector` | Create connectors (GitHub, AWS, GCP, Azure, Docker, K8s) |
| `/create-secret` | Manage secrets (SecretText, SecretFile, SSHKey, WinRM) |

### Access Control & Users

| Skill | Description |
|-------|-------------|
| `/manage-users` | Manage users, user groups, and service accounts |
| `/manage-roles` | RBAC roles, assignments, permissions, and resource groups |

### Feature Flags

| Skill | Description |
|-------|-------------|
| `/manage-feature-flags` | Create, list, toggle, and delete feature flags |

### Platform Operations

| Skill | Description |
|-------|-------------|
| `/manage-delegates` | Monitor delegate health and manage registration tokens |

### Observability & Governance

| Skill | Description |
|-------|-------------|
| `/analyze-costs` | Cloud cost analysis, recommendations, and anomaly detection |
| `/security-report` | Security vulnerabilities, SBOMs, and compliance reports |
| `/dora-metrics` | DORA metrics and engineering performance reports |
| `/gitops-status` | GitOps application health, sync status, and pod logs |
| `/chaos-experiment` | Create and run chaos engineering experiments |
| `/scorecard-review` | IDP scorecards and service maturity review |
| `/audit-report` | Audit trails and compliance evidence (SOC2, GDPR, HIPAA) |
| `/template-usage` | Template dependency tracking, impact analysis, and adoption |
| `/create-policy` | Create OPA governance policies for supply chain security |

### Agents

| Skill | Description |
|-------|-------------|
| `/create-agent-template` | Generate AI agent templates (metadata.json, pipeline.yaml, wiki.MD) |

## Cross-Skill Workflows

When users need end-to-end setup, follow these dependency chains. Each step depends on the previous -- do not skip steps or create resources that reference connectors/secrets that don't exist yet.

### New Microservice Setup

Use when a user wants to deploy a new service to Harness. Follow this exact order:

1. **Create connectors** (`/create-connector`) -- GitHub connector for source code and manifests, Docker/ECR/GCR connector for container images, K8s/cloud connector for target infrastructure
2. **Create secrets** (`/create-secret`) -- Authentication tokens, SSH keys, or credentials referenced by connectors
3. **Create service** (`/create-service`) -- Reference the Git connector for manifests, reference the Docker connector for artifact source
4. **Create environment** (`/create-environment`) -- PreProduction and/or Production with environment-specific variables
5. **Create infrastructure** (`/create-infrastructure`) -- Reference the K8s/cloud connector for the target cluster. If no connector exists for the target, create one first
6. **Create pipeline** (`/create-pipeline`) -- Build pipeline based on source code language and manifest type. Reference the service, environment, and infrastructure from previous steps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harness/harness-skills](https://github.com/harness/harness-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
