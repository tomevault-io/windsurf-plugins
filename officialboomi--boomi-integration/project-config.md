---
trigger: always_on
description: Develops and deploys Boomi integrations, APIs, and platform services including Event Streams, Flow Services, and web service endpoints. Use when building, modifying, or deploying Boomi components (processes, profiles, connections, operations, maps, API components, EDI, Flow Services). Handles bi-directional sync, deployment automation, testing, and provides comprehensive Boomi reference documentation for complete solution development.
---


# Implementing Boomi with this Skill
This is the Boomi Process Development Framework - a reusable skill that enables AI coding agents to build Boomi integration processes programmatically. It provides CLI tools, reference documentation, and patterns for bi-directional synchronization with the Boomi platform API.

**Architecture**: The framework is separated from project components:
- **boomi-integration skill**: Reusable infrastructure, tools, documentation
- **active-development/** (project root): All working files - components, sync state, feedback

**Running CLI tools — resolve `<skill-path>` at the beginning of each session.** All script invocations in this skill use the placeholder `<skill-path>/scripts/...`. Reuse the result for every later call:

- [ ] Take the absolute path of this SKILL.md you just read and drop the trailing `/SKILL.md`. That directory is `<skill-path>`.
- [ ] Verify by running `bash <skill-path>/scripts/boomi-env-check.sh`. If bash reports "No such file," stop and re-locate the skill rather than guessing the path. (Doubles as your `.env` status check — see § First-Time User Detection.)
- [ ] Once resolved, treat `<skill-path>` as a fixed value for the session. Every subsequent invocation must re-emit the same path string verbatim — drift between calls (e.g. dropping `skills/boomi-integration/`) produces "No such file" errors even when the first call worked.

Run scripts from the project workspace directory (so `.env` and `active-development/` resolve correctly), but always invoke them with the full absolute `<skill-path>/scripts/...` path.

## Documentation Architecture

**SKILL.md is the navigation hub**: This file contains file references and routing guidance. Other documentation files contain minimal cross-references by design - this prevents deep hierarchical dependencies (which causes skimming at lower levels) and keeps navigation centralized.

**Complex tasks require multiple files**: Most Boomi development tasks require consulting 3-5+ documentation files together. The "Use when" guidance throughout this file indicates which combinations to load for specific scenarios.

**Common multi-file workflows:**
- **Adding any step**: BOOMI_THINKING.md + process_component.md + steps/[step].md + dependency component docs
- **Creating connectors**: connection_component.md + operation_component.md + connector_step.md + BOOMI_THINKING.md
- **REST API on Advanced atom**: api_service_component.md + web_services_server_start_shape_operation.md + process_component.md + api_conversion_patterns.md
- **Map transformations**: map_component.md + map_component_functions.md + source/target profile docs
- **Event Streams**: event_streams_connection + operation + steps + platform_entities/event_streams.md
- **B2B/EDI Trading Partners**: trading_partner_component.md + trading_partner_steps.md + edi_profile_component.md + platform_entities/edi_b2b.md
- **Disk V2 (File System)**: diskv2_connection_component + diskv2_connector_operation_component + diskv2_connector_step
- **MFT (Managed File Transfer)**: mft_connection_component + mft_connector_operation_component + mft_connector_step
- **Mail (IMAP) — Email send/receive/move**: mail_imap_connection_component + mail_imap_connector_operation_component + mail_imap_connector_step + (document_cache_component for attachments)
- **MCP Server (AI Tool Exposure)**: mcp_server_connection_component + mcp_server_operation_component + mcp_server_start_step + platform_entities/mcp_server.md
- **Agent Step (AI Agent in process)**: agent_step.md
- **Flow Services**: fss_operation_component + fss_start_step + flow_service_component + platform_entities/flow.md
- **Debugging**: boomi_error_reference.md + relevant step/component docs
- **Branch & Merge** (opt-in only): branch_merge_guide.md + cli_tool_reference.md branch workflows section
- **Version management**: version_management_guide.md + cli_tool_reference.md version management section

## First-Time User Detection
§ Running CLI tools already verifies `<skill-path>` and reports `.env` status via `boomi-env-check.sh`. If `bash <skill-path>/scripts/boomi-folder-create.sh --test-connection` fails, route the user through `references/guides/user_onboarding_guide.md` or `/bc-integration:env-setup-guide`.

## Connection Discovery & Credential Security
**Connection re-use is recommended.** Pulling existing connections keeps credentials out of the conversation. Offer the connection discovery workflow first, but respect the user's choice if they prefer to provide credentials directly. See `references/BOOMI_THINKING.md` § Connection Discovery for the full workflow.

## Workspace Organization & Knowledge Base
### Physical Directory Structure & Documentation Inventory
**Core Mental Models:**
- `references/BOOMI_THINKING.md` - Core mental models and development philosophy (always read first)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OfficialBoomi/boomi-integration](https://github.com/OfficialBoomi/boomi-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
