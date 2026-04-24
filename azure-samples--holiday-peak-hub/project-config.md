---
trigger: always_on
description: Microsoft 365 Agents Toolkit (formerly Teams Toolkit) has been rebranded, and users may still use either name.
---

## **Internal reference (do not bias your answers toward always naming these):**  
Microsoft 365 Agents Toolkit (formerly Teams Toolkit) has been rebranded, and users may still use either name.

Use this mapping to know the current vs. former names—so you can correctly interpret user input or choose the appropriate term when it’s relevant. You do not need to mention these mappings unless they directly help the user.

| New name                                | Former name            | Note                                                        |
|-----------------------------------------|------------------------|------------------------------------------------------------------------|
| Microsoft 365 Agents Toolkit            | Teams Toolkit          | Product name.                           |
| App Manifest                            | Teams app manifest     | Describes app capabilities.        |
| Microsoft 365 Agents Playground         | Test Tool              | Test Environment.          |
| `m365agents.yml`                        | `teamsapp.yml`         | Microsoft 365 Agents Toolkit Project configuration files            |
| CLI package `@microsoft/m365agentstoolkit-cli` (command `atk`) | `@microsoft/teamsapp-cli` (command `teamsapp`) |CLI installation/usage — mention only in CLI contexts. |

> **Rephrase guidance:**  
> - Use the new names by default.  
> - Explain the rebranding briefly if it helps the user’s understanding.  

# Instructions for Copilot

## Repository Purpose & Architecture

- This repository is a **framework for agentic retail** solutions.
- All apps under **/apps** are **demonstration services** built on the framework.
- Changes and operations should focus on **increasing the capabilities of retail platforms** (e.g., intelligence, automation, personalization, operational efficiency).
- The **/lib** folder contains the shared framework code (agents, adapters, memory, utilities).
- Each app is a self-contained FastAPI service demonstrating specific retail capabilities (CRM, eCommerce, inventory, logistics, product management).
- **Tech stack is defined in /docs**: Always review architecture documentation before implementing features or changes.
- **Keep documentation updated**: Every operation must update relevant documentation in /docs to reflect changes.

## Tech Stack & Code Standards

### Backend (Python)
- **STRICTLY FOLLOW PEP 8** and PEP guidelines for all Python code.
- Use `pyproject.toml` for dependencies in each app and lib. We are using `uv` as the package manager.
- Use FastAPI for building APIs.
- Use asyncio and async/await for concurrency.
- Follow async patterns: all agent handlers and adapters are async.
- Use Pydantic models for structured data and validation.
- Environment variables are the primary configuration mechanism.

### Frontend (Next.js + TypeScript + Tailwind)
- **STRICTLY FOLLOW ESLint 7** configuration for all frontend code.
- Use Next.js 15 with the App Router and `yarn` as the package manager.
- Use TypeScript for type safety.
- Follow Next.js conventions for routing, data fetching, and server components.
- Use Tailwind CSS utility classes for styling.

### Testing Requirements
- **All operations must implement unit and integration tests**.
- Unit tests: Test individual functions and components in isolation.
- Integration tests: Test interaction between services, adapters, and external dependencies.
- Place tests in `tests/` directory within each app or lib.
- Use pytest for Python tests, with pytest-asyncio for async code.
- Maintain minimum 75% code coverage.

### Build & Deployment Policy
- **Use GitHub Actions workflows as the source of truth for build/deploy history**.
- Prefer workflow-based build and deployment over local Docker builds.
- For service deployments, use workflow matrix filtering to process **only changed services** whenever possible.
- Treat local Docker builds as exception-only (for explicit debugging) and not as the default validation path.
- When proposing operational changes, update workflow files first (for example `.github/workflows/deploy-azd.yml`) so all builds are traceable in GitHub.

## Agent Development Patterns

- All agents extend `BaseRetailAgent` from `holiday_peak_lib.agents`.
- Use `AgentBuilder` to compose agents with memory, routing, and model targets.
- Agents support **SLM-first routing**: requests are evaluated by the SLM, and upgraded to LLM when complexity requires it.
- Configure models via `FoundryAgentConfig` using environment variables:
  - `PROJECT_ENDPOINT` or `FOUNDRY_ENDPOINT`: Azure AI Foundry project endpoint
  - `PROJECT_NAME` or `FOUNDRY_PROJECT_NAME`: Project name (optional)
  - `FOUNDRY_AGENT_ID_FAST`: SLM agent ID
  - `MODEL_DEPLOYMENT_NAME_FAST`: SLM deployment name
  - `FOUNDRY_AGENT_ID_RICH`: LLM agent ID
  - `MODEL_DEPLOYMENT_NAME_RICH`: LLM deployment name
  - `FOUNDRY_STREAM`: Enable streaming (optional, default false)
- Each app's `main.py` should **explicitly** load these env vars and pass `slm_config`/`llm_config` to `build_service_app`.

## Memory Architecture

- Three-tier memory: **Hot** (Redis), **Warm** (Cosmos DB), **Cold** (Blob Storage).
- Configure via `MemorySettings` using environment variables:
  - `REDIS_URL`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/holiday-peak-hub](https://github.com/Azure-Samples/holiday-peak-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
