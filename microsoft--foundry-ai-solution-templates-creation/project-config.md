---
trigger: always_on
description: Scaffold a complete, production-ready Azure project from scratch for ANY use case: RAG chatbots, multi-agent systems, API backends, data pipelines, Azure Functions, full-stack web apps, ML training/inference, or event-driven microservices. USE FOR: gathering requirements through adaptive questioning, generating complete project folder structures with all source files, Azure Bicep infrastructure, azure.yaml for azd-up deployment, Docker configuration, CI/CD workflows, README with architecture dia
---


# Azure Scaffold Wizard — Universal Project Generator

This skill generates a **complete, production-ready Azure project** deployable with `azd up`. It is fully domain-agnostic and applies to any use case. It produces every file needed: requirements documentation, project structure with all source files, README, Azure Bicep infrastructure, `azure.yaml`, Docker configuration, CI/CD workflows, architecture diagrams, observability setup, and Responsible AI documentation.

---

## Required Workflow

Follow these steps **in order**. Do not skip or reorder steps. Each step's output feeds the next.

---

### Step 1 — Identify Project Type and Gather Requirements

This is the most critical step. **Do not generate any files until this step is complete.**

#### 1A — Project Type Selection

Ask the user: **"What type of Azure project do you want to build?"**

Present these options:

| # | Project Type | Description | When to Choose |
|---|---|---|---|
| 1 | **RAG Chatbot** | Retrieval-Augmented Generation chatbot with vector search | Conversational AI grounded in your own data |
| 2 | **Multi-Agent System** | Multiple AI agents coordinated by an orchestrator | Parallel/sequential AI reasoning pipeline (Foundry compatible) |
| 3 | **API Backend** | REST or GraphQL API service | Backend service with database and Azure deployment |
| 4 | **Data Pipeline** | ETL/ELT data processing pipeline | Batch or streaming data transformation |
| 5 | **Azure Functions** | Serverless event-driven functions | Lightweight event triggers and integrations |
| 6 | **Full-Stack Web App** | Frontend + Backend web application | Complete web application with UI |
| 7 | **ML Training & Inference** | Model training and serving endpoint | Custom ML model lifecycle on Azure ML |
| 8 | **Event-Driven Microservices** | Message-based distributed system | Async decoupled services at scale |

If the user's request does not fit these categories exactly, choose the closest match or combine patterns. For detailed descriptions, read `references/project-types.md`.

#### 1B — Universal Requirements (ask for ALL project types)

| # | Question | Guidance |
|---|---|---|
| U1 | **What business problem does this solve?** | One sentence. Drives README opening and all documentation. |
| U2 | **Who are the end users?** | Drives UI design, access control decisions, RBAC roles. |
| U3 | **What is the project name?** | kebab-case slug for folder names, azure.yaml, and resource naming. |
| U4 | **Programming language?** | Python (default), TypeScript, C#, or multi-language. Drives all source file templates. |
| U5 | **Azure region?** | Default: `eastus2`. Constrain by AI model availability if applicable. |
| U6 | **Authentication required?** | None (default scaffold), Microsoft Entra ID (production). |
| U7 | **What Azure services beyond the defaults?** | e.g., Cosmos DB, Service Bus, Key Vault, Storage, Redis. Drives extra Bicep modules. |
| U8 | **Frontend required?** | Yes/No. If yes: Next.js (default), React SPA, or other. |
| U9 | **What compliance or regulatory requirements?** | e.g., HIPAA, SOC2, GDPR. Drives TRANSPARENCY_FAQ.md and security docs. |
| U10 | **Target deployment platform?** | Azure Container Apps (default), Azure Functions, AKS, App Service. |
| U11 | **Use Azure AI Foundry Agent Service?** | **Yes** (default for AI-capable types): AI processing via Foundry Hosted Agents with structured output, managed deployment, and MAF. **No**: hand-rolled code using Azure OpenAI SDK directly. **When to ask**: Type 1 (RAG) — always ask. Types 3, 4, 5, 6, 8 — ask only after the type-specific "Include AI?" question is answered Yes. **Skip for**: Type 2 (Multi-Agent, always Foundry) and Type 7 (ML Training, uses Azure ML). |

#### 1C — Type-Specific Requirements

Based on the project type selected in 1A, read the corresponding reference file and ask the type-specific questions defined there:

| Project Type | Reference File | What It Adds |
|---|---|---|
| RAG Chatbot | `references/rag-chatbot.md` | Vector store, embedding model, chunking strategy, data sources, conversation memory |
| Multi-Agent System | `references/multi-agent.md` | Agent names/roles, orchestration topology, MCP tools, Foundry model choice |
| API Backend | `references/api-backend.md` | Endpoints, database, API style (REST/GraphQL), rate limiting |
| Data Pipeline | `references/data-pipeline.md` | Data sources/sinks, scheduling, batch vs. stream, transformations |
| Azure Functions | `references/function-app.md` | Triggers, bindings, Durable Functions, runtime stack |
| Full-Stack Web App | `references/full-stack-app.md` | Frontend framework, SSR/SPA, state management, API integration |
| ML Training & Inference | `references/ml-training.md` | ML framework, compute targets, model registry, managed endpoints |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Foundry-AI-solution-templates-creation](https://github.com/microsoft/Foundry-AI-solution-templates-creation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
