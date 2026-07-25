---
trigger: always_on
description: This extension connects Gemini CLI to the Harness Platform through 11 consolidated MCP tools that cover 202 resource types across 33 default toolsets.
---

# Harness MCP Server — Gemini CLI Context

This extension connects Gemini CLI to the Harness Platform through 11 consolidated MCP tools that cover 202 resource types across 33 default toolsets.

## How This Server Works

Unlike traditional MCP servers with one tool per API endpoint, this server uses a **registry-based dispatch** pattern. You interact through generic verb-based tools and specify the `resource_type` you want to work with.

**Start with discovery:**
- `harness_describe` — Browse all available resource types (no API call, instant)
- `harness_describe` with `search_term` — Find resource types by keyword
- `harness_describe` with `resource_type` — Get full detail on a specific type

**Then use CRUD tools:**
- `harness_list` — List resources with filtering and pagination
- `harness_get` — Get a single resource by ID
- `harness_create` — Create a resource (elicits confirmation when supported by the MCP client)
- `harness_update` — Update a resource (elicits confirmation when supported by the MCP client)
- `harness_delete` — Delete a resource (destructive; blocked if confirmation cannot be obtained)

**Specialized tools:**
- `harness_execute` — Run pipelines, toggle feature flags, test connectors, sync GitOps apps; pass `wait: true` for server-side pipeline run/retry polling
- `harness_search` — Search across multiple resource types at once
- `harness_diagnose` — Diagnose pipelines, connectors, delegates, and GitOps applications
- `harness_status` — Project health overview: failed, running, and recent executions
- `harness_schema` — Fetch bundled pipeline/template schemas, named YAML examples, and scope-aware entity schemas for connectors, environments, services, secrets, and infrastructure

`harness_list` returns strict structured content for MCP clients: array-like Harness responses are normalized into `{ "items": [...], "total": <count>, "page": <page> }`, while the text payload still contains compact JSON.

## Available Capabilities

### CI/CD & Pipelines
- List, view, create, update, and delete pipelines
- Execute pipelines with runtime inputs, retry failed executions, interrupt running ones
- View execution history and download execution logs
- Manage pipeline triggers and input sets
- Pipeline run shorthand support: `branch`, `tag`, `pr_number`, and `commit_sha` auto-expand into CI build input structures (unless `inputs.build` is already provided explicitly)
- Pipeline run/retry wait mode returns terminal status fields or `_wait` recheck hints without requiring a client-side polling loop

### Services & Environments
- CRUD operations on services and environments
- Manage infrastructure definitions
- Move environment and infrastructure configs between scopes

### Connectors & Secrets
- List, create, update, and test connectors
- Browse the connector catalogue
- View secret metadata (values are never exposed)

### Cloud Cost Management (CCM)
- Analyze costs with perspectives, breakdowns, and time series
- Access optimization recommendations with savings estimates
- Detect and manage cost anomalies
- Track commitment coverage, utilisation, and savings

### Security & Compliance
- Security Test Orchestration (STO): manage issues, create exemptions, and approve or reject exemptions with explicit approval scope
- Supply Chain Security (SCS): track artifacts, compliance, SBOMs, chain of custody
- Audit trail: registry-dispatched list/get/create/update/delete/execute operations can emit structured events through the default logger-filtered stderr sink, optional durable JSONL/webhook sinks, and optional OpenTelemetry spans

### GitOps
- Manage agents, applications, clusters, repositories
- Sync applications, view resource trees, access pod logs
- Track application events and managed resources

### Chaos Engineering
- List and run chaos experiments
- Create experiments from templates
- View experiment run results and probe details
- Manage load tests

### Feature Flags
- List and manage feature flags across environments
- Toggle flags on/off with environment targeting

### Internal Developer Portal (IDP)
- Manage catalog entities and scorecards
- Track developer experience scores and checks
- Execute IDP workflows, search tech docs

### Templates & Dashboards
- Browse and use pipeline, stage, and step templates
- Access custom dashboards and data exports

### Infrastructure as Code Management (IaCM)
- Default-enabled toolset for Terraform workspaces, resources, module registry entries, workspace costs, and activity resource changes
- Workspace, resource, cost, and activity-change APIs require org/project scope; the module registry is account-scoped
- Use `iacm_workspace` to find `workspace_id`; `iacm_activity_resource_change` requires both `activity_id` and `workspace_id`
- IaCM `page_count` values count the current page only; paginate while `has_more` is true when a total is needed

### Ansible
- Opt-in toolset for Ansible inventories, playbooks, hosts, and activity history
- Enable with `HARNESS_TOOLSETS=+ansible`; Ansible APIs require org/project scope

### Database DevOps (DbOps)
- Manage database schemas and instances with create/update/delete support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harness/mcp-server](https://github.com/harness/mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
