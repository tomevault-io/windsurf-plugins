---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

This is a Terraform provider built with [HashiCorp Terraform Plugin Framework](https://developer.hashicorp.com/terraform/plugin/framework) v1.13.0.

- `main.go` — entry point; serves the provider at `registry.terraform.io/ippontech/anthropic`
- `internal/provider/provider.go` — provider registration; `Resources()` and `DataSources()` methods list all implemented resources and data sources
- `internal/services/` — all resources and data sources, organized by Anthropic service (one subdirectory per service)
- `examples/provider/` — example Terraform configs used by `terraform-plugin-docs` to generate `docs/`
- `tests/` — Terraform native tests (`.tftest.hcl` files), one per resource/data source
- `tools/tools.go` — build-time tool imports only (not runtime)

### Internal package layout

```
internal/
  admin/           — HTTP client for Admin API (/v1/organizations/*)
  acctest/         — shared acceptance test helpers (ProtoV6ProviderFactories, PreCheck)
  errors/          — nil-client guards for Configure methods (import alias: providerrors)
  providerdata/    — ProviderData struct passed to every resource/data source Configure call
  retry/           — multipart upload with 5xx retry (import alias: provretry)
  provider/        — AnthropicProvider implementation only (provider.go)
  services/
    agents/        — anthropic_agent resource + agent/agents data sources
    apikeys/       — anthropic_api_key resource (import/update/delete only; no create) + api_key/api_keys data sources
    environments/  — anthropic_environment resource + environment/environments data sources
    messages/      — anthropic_message resource + count_tokens data source
    models/        — model/models data sources
    skills/        — skill/skill_version resources + skill/skills/skill_version/skill_versions data sources
    workspaces/    — anthropic_workspace + anthropic_workspace_member resources + workspace/workspaces/workspace_member/workspace_members data sources (shared test helpers in workspacetest.go)
```

### Implemented resources and data sources

**Resources:**
- `anthropic_message` (`internal/services/messages/message_resource.go`) — calls the Messages API; write-only, immutable (no read/update/delete)
- `anthropic_agent` (`internal/services/agents/agent_resource.go`) — manages Managed Agents (create/read/update/delete)
- `anthropic_environment` (`internal/services/environments/environment_resource.go`) — manages environments; supports `archive_on_destroy` (archives instead of deleting on destroy when true)
- `anthropic_skill` (`internal/services/skills/skill_resource.go`) — manages skills
- `anthropic_skill_version` (`internal/services/skills/skill_version_resource.go`) — manages skill versions
- `anthropic_workspace` (`internal/services/workspaces/workspace_resource.go`) — manages workspaces (admin API)
- `anthropic_workspace_member` (`internal/services/workspaces/workspace_member_resource.go`) — assigns a user to a workspace with a given role (admin API); composite ID `<workspace_id>:<user_id>`; `workspace_billing` role rejected at plan time
- `anthropic_api_key` (`internal/services/apikeys/api_key_resource.go`) — import-only resource; manages lifecycle of existing API keys (rename, deactivate) via Admin API; Create always errors with a message to use `terraform import`; Delete sets `status: inactive`

**Data sources:**
- `anthropic_model` (`internal/services/models/model_data_source.go`) — fetches a single model by ID
- `anthropic_models` (`internal/services/models/models_data_source.go`) — lists all available models
- `anthropic_count_tokens` (`internal/services/messages/count_tokens_data_source.go`) — counts tokens for a given prompt
- `anthropic_agent` (`internal/services/agents/agent_data_source.go`) — fetches a single agent
- `anthropic_agents` (`internal/services/agents/agents_data_source.go`) — lists all agents
- `anthropic_environment` (`internal/services/environments/environment_data_source.go`) — fetches a single environment
- `anthropic_environments` (`internal/services/environments/environments_data_source.go`) — lists all environments
- `anthropic_skill` (`internal/services/skills/skill_data_source.go`) — fetches a single skill
- `anthropic_skills` (`internal/services/skills/skills_data_source.go`) — lists all skills
- `anthropic_skill_version` (`internal/services/skills/skill_version_data_source.go`) — fetches a single skill version
- `anthropic_skill_versions` (`internal/services/skills/skill_versions_data_source.go`) — lists all skill versions
- `anthropic_workspace` (`internal/services/workspaces/workspace_data_source.go`) — fetches a single workspace by ID (admin API)
- `anthropic_workspaces` (`internal/services/workspaces/workspaces_data_source.go`) — lists all workspaces (admin API, transparent pagination)
- `anthropic_workspace_member` (`internal/services/workspaces/workspace_member_data_source.go`) — fetches a single workspace member by workspace ID and user ID (admin API)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ippontech/terraform-provider-anthropic](https://github.com/ippontech/terraform-provider-anthropic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
