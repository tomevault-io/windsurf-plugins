---
trigger: always_on
description: All Copilot changes MUST comply with the Home Assistant Integration Quality Scale as defined in `ha-developer-docs/core/integration-quality-scale` and its linked rules.
---

# Integration Quality Scale Compliance

All Copilot changes MUST comply with the Home Assistant Integration Quality Scale as defined in `ha-developer-docs/core/integration-quality-scale` and its linked rules.

- Never introduce changes that would lower the integration’s quality tier or violate any mandatory rule.
- Always aim to raise the integration’s quality tier by:
    - Improving test coverage and reliability.
    - Enhancing user experience, documentation, and diagnostics.
    - Ensuring robust error handling and reauthentication.
    - Refactoring for async operation, strict typing, and code clarity.
    - Supporting translations, logical naming, and discoverability.
- Before making changes, check the current tier and rules in `ha-developer-docs/core/integration-quality-scale/`.
- If a rule or requirement is unclear, consult the linked rule documentation and err on the side of higher quality.
- Document any improvements toward a higher tier in the PR or commit message.
- Never remove or bypass automated tests, diagnostics, or codeowner requirements.
- All new features and refactors must move the integration closer to Gold or Platinum, unless explicitly instructed otherwise.

#### Quality Tiers (Summary)

- **Bronze:** UI config flow, unique entity IDs, automated tests, codeowners, basic docs.
- **Silver:** All Bronze + robust error handling, stable UX, troubleshooting docs, auto recovery.
- **Gold:** All Silver + best UX, auto discovery, translations, diagnostics, >95% test coverage, firmware updates if possible.
- **Platinum:** All Gold + supreme code quality, fully async, strict typing, efficient data handling.
# Agent Instructions - Home Assistant Integration Development

## Overview
This project is a Home Assistant integration for TP-Link Omada Open API.

**Purpose**: Provide information retrieved via the Omada Open API as entities within Home Assistant, enabling monitoring and control of Omada SDN infrastructure (controllers, access points, switches, gateways, and clients).

## Documentation References

### CRITICAL: Documentation Hierarchy
**ALWAYS** consult official documentation in GitHub repos before making any changes:


1. **Home Assistant Developer Documentation (Local)**
    - **Primary Source:** Use the local `ha-developer-docs` folder in this repository as the authoritative source for all Home Assistant integration development details.
    - Covers: Integration patterns, best practices, quality standards, entity types, config flows, and more.
    - **MUST** be followed strictly for all HA integration development.
    - **Note:** This folder is intentionally .gitignored but always present in the dev environment for agent use.

2. **If information is not found in `ha-developer-docs`, fallback to the official GitHub documentation:**
    - Use GitHub MCP tool to search: `home-assistant/developers.home-assistant`
    - Use only if the local docs do not provide the needed details.
    - Covers: Integration patterns, best practices, quality standards, entity types, config flows, etc.
    - **MUST** be followed strictly for all HA integration development

2. **HACS Publishing Documentation**
   - Use GitHub MCP tool to search: `hacs/documentation` (specifically `source/docs/publish/`)
   - Authoritative source for: HACS requirements, repository structure, validation, publishing workflow
   - **MUST** be followed strictly when preparing for HACS publication

3. **Omada Open API Documentation**
   - **Official Documentation**: https://use1-omada-northbound.tplinkcloud.com/doc.html#/home
   - **OpenAPI Specification**: `/workspaces/ha-omada-open-api/openapi/openapi.json` (complete Swagger/OpenAPI spec)
   - Authoritative source for: All endpoint definitions, parameters, request/response schemas, data models


**When there are doubts or questions**:
- **Home Assistant patterns**: Consult the local `ha-developer-docs` folder FIRST. If not found, search `home-assistant/developers.home-assistant` on GitHub.
- **HACS requirements**: Search `hacs/documentation` (source/docs/publish/) FIRST
- **Omada API**: Consult `openapi/openapi.json` FIRST


**Enforcement**:
- Always consult the local `ha-developer-docs` folder before proposing or implementing changes that touch Home Assistant patterns, config flows, entities, services, or HACS publishing requirements.
- If no relevant documentation is found locally, search the official GitHub documentation as a fallback.
- If still no relevant documentation is found, say so explicitly and continue with best-effort reasoning.

## Omada Open API Integration Specifics

### API Architecture
- **Cloud-based API**: Omada Controller accessed via TP-Link cloud infrastructure
- **RESTful API**: Standard REST endpoints for data retrieval and device management
- **Authentication**: OAuth 2.0 flow with access tokens and refresh tokens
- **Base URL**: Region-specific endpoints (e.g., use1-omada-northbound.tplinkcloud.com)

### Authentication & Session Management
- **OAuth Flow**: OAuth 2.0 Client Credentials grant type (not Authorization Code)
- **Token Lifecycle**: Access tokens expire in 2 hours, refresh tokens in 14 days

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bullitt186/ha-omada-open-api](https://github.com/bullitt186/ha-omada-open-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
