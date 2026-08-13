---
trigger: always_on
description: Refer to [docs/PRODUCT.md](docs/PRODUCT.md) for the complete Product Requirement Document containing the product vision, functional specifications, target audience, and success metrics.
---

# FinSavant

## Project Goals and Requirements

Refer to [docs/PRODUCT.md](docs/PRODUCT.md) for the complete Product Requirement Document containing the product vision, functional specifications, target audience, and success metrics.

## Tool Use: Skills, Gemini Enterprise Agent Platform, Agent Runtime and ADK

Be sure to use all **agents** skills, **Gemini Enterprise Agent Platform** skills, and **ADK** skills you have available for developing ADK agents and best practices, and use **adk-docs-mcp** for latest ADK documentation. 

You will have additional skills available to you, but always check if the following can help with a particular task.

### ADK & agents-cli Lifecycle Skills

- `google-agents-cli-workflow`: Entrypoint for building ADK agents (scaffold, build, evaluate, deploy, publish, observe).
- `google-agents-cli-scaffold`: Creating and upgrading agent projects (agents-cli scaffold create/enhance/upgrade).
- `google-agents-cli-adk-code`: Agent Development Kit (ADK) Python API patterns, tool definitions, callbacks, and state management.
- `google-agents-cli-deploy`: Configuring and executing deployments to Agent Runtime, Cloud Run, or GKE.
- `google-agents-cli-eval`: Running agent evaluations and understanding the Agent Platform Quality Flywheel.
- `google-agents-cli-observability`: Monitoring, tracing, and logging deployed ADK agents in production.
- `google-agents-cli-publish`: Registering and publishing ADK agents to the Gemini Enterprise / Agent Registry.

### Gemini Enterprise Agent Platform APIs

- `gemini-api`: Vertex AI, Google Cloud, and Agent Platform enterprise usage with the Google Gen AI SDK.
- `gemini-agents-api / gemini-managed-agents-api`: Creating, configuring, and managing custom Agent resources programmatically.
- `gemini-interactions-api`: Stateful, server-managed multi-turn conversation and function execution workflows.

### Agent Platform Engine & Model Management

- `agent-platform-deploy`: Deploying models and tuned weights to Agent Platform endpoints.
- `agent-platform-model-registry`: Uploading, versioning, and managing models in the Agent Platform Model Registry.
- `agent-platform-prompt-management`: Managing and versioning system/agent prompts.
- `agent-platform-rag-engine-management`: Managing RAG Engine Corpora and retrieving grounded contexts.
- `agent-platform-skill-registry`: Integrating and searching for registered agent skills.
- `agent-platform-tuning`: Fine-tuning models on Agent Platform infrastructure.
- `agent-platform-tuning-management`: Managing GenAI tuning jobs (listing, checking, cancelling).

## ADK + React UI Best Practices

- **Backend for Frontend (BFF)**: Use FastAPI as a thin layer to serve static React assets and provide a robust API for the ADK agent.
- **Rich UI (A2UI)**: Leverage the **Agent-to-UI (A2UI)** protocol for structured data outputs. The agent should return `application/json+a2ui` payloads for complex components like tables, charts, and cards.
- **UI Acceleration**: Use **Stitch with MCP** to rapidly build and iterate on information-dense dashboards.

## Key Internal Documentation

- README.md - Project README; the developer's front door
- TODO.md - High level plan for the project
- docs/PRODUCT.md - The product spec
- docs/architecture-and-walkthrough.md - The main architecture, including design decisions
- docs/spec_multiagent_split.md - Technical specification for the multi-agent split phase
- docs/DESIGN.md - Where we will capture the UI design
- docs/testing.md - Where we will document test strategy, summary of tests, testing instructions, any manual testing processes
- docs/blog.md - A blog post document we will build along the way
- /deployment/README.md - Deployment and CI/CD documentation

## Essential Reading

You should read and leverage these resources for guidance and best practices, in addition to the skills and MCP servers you have available for knowledge.

| Resource | Description and Relevance |
| -------- | ------------------------- |
| https://docs.cloud.google.com/bigquery/docs/use-bigquery-mcp | Use the BigQuery MCP server | 
| https://adk.dev/integrations/bigquery/ | BigQuery tool for ADK |
| https://docs.cloud.google.com/gemini-enterprise-agent-platform | Gemini Enterprise Agent Platform Overview |
| https://adk.dev/deploy/agent-runtime | ADK with Agent Runtime |
| https://adk.dev/deploy/agent-runtime/deploy/ | Deploying ADK agents to Agent Runtime |
| https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/runtime/quickstart-adk | Agent Runtime Quickstart |
| https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/runtime/deploy-an-agent#from-source-files | Deploying to Agent Runtime |
| https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/observability/overview | Gemini Enterprise Agent Platform - Observability Overview |
| https://docs.cloud.google.com/asset-inventory/docs/asset-inventory-overview | Google Cloud Asset Inventory overview. This provides an overview of Google Cloud Asset Inventory, and how to use it. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derailed-dash/smart-gcp-finops](https://github.com/derailed-dash/smart-gcp-finops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
