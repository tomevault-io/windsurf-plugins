---
trigger: always_on
description: >
---


# Oracle AI Agent Studio

A comprehensive skill for working with Oracle AI Agent Studio — the design-time environment
within Oracle Fusion Cloud Applications for creating, configuring, validating, and deploying
AI agents and agent teams across the enterprise.

## When to Use This Skill

- Creating new AI agents or agent teams in Oracle Fusion
- Customizing or extending pre-built Oracle agent templates
- Configuring agent tools (Document, REST, Business Object, Calculator, Email)
- Setting up multi-agent workflows with branching, chaining, and human-in-the-loop
- Connecting agents to third-party LLMs or external systems via MCP
- Testing, evaluating, and monitoring agent performance
- Deploying agents through the Oracle AI Agent Marketplace
- Integrating custom agents into Fusion Service, HCM, ERP, or SCM workflows

For Oracle Database-specific skills (SQL tuning, PL/SQL, ORDS, security audits), see
`krisrice/oracle-db-skills` instead. This skill focuses on the Fusion Applications
agent platform, not the OCI AI Database Private Agent Factory (see `references/oci-agent-factory.md`
for that).

---

## Architecture Overview

Oracle AI Agent Studio has five core components that work together hierarchically:

```
Agent Team (deployable unit)
 └── Agent(s)
      ├── Topics (expertise boundaries)
      ├── Instructions (natural language rules per topic)
      └── Tools (utilities the agent can invoke)
```

### Component Details

**Agent Team**: The deployable unit. Consists of a structured sequence of steps or actions
that one or more agents follow to accomplish a business task. Agent teams define conversation
logic, system integration, and user support flow. An agent team is what gets published and
invoked at runtime.

**Agent**: Uses an LLM to reason, plan, and interact with users. Must belong to an agent team.
Three types exist:
- *User-proxy agent*: Acts on behalf of a user, gathers input, provides conversational interface
- *Supervisor agent*: Orchestrates multiple agents within a workflow
- *Specialist/utility agent*: Focused on a specific role or tool expertise

Agents can also be persona-based (e.g., benefits administrator), tool users (calculators,
document retrieval), or task-oriented (single step in a multi-agent flow).

**Topics**: Define expertise boundaries through instructions. Example: an employee benefits
agent might have topics for HSA, retirement benefits, and stock plans. Topics set the scope
of what an agent can and cannot discuss.

**Tools**: Additional utilities an agent can invoke. Types include:
- *Document tool*: For RAG — upload documents the agent can search
- *Business Object tool*: Query/update Fusion business objects
- *REST tool*: Call external HTTPS APIs
- *Calculator tool*: Perform computations
- *Email tool*: Send notifications
- *User query tool*: Interactive data lookup
- *Agent tool*: Invoke another agent (for multi-agent collaboration)
- *MCP tool*: Connect to external systems via Model Context Protocol

**Instructions**: Natural language rules per topic. Become part of the prompt sent to the LLM.
Include guidelines, guardrails, and response parameters.

---

## Getting Started

### Prerequisites

1. Oracle Fusion Cloud Applications environment (25A or later for full Agent Studio)
2. Appropriate roles granted by an AI Studio Administrator
3. `ORA_ASE_SAS_INTEGRATION_ENABLED` profile option set to `Yes`
4. Permission groups enabled for the appropriate roles

For full access requirements, read `references/access-requirements.md`.

### Navigating to AI Agent Studio

Navigate to: **Navigator > Tools > AI Agent Studio**

The interface has four main tabs:
- **Tools** — Create and manage agent tools
- **Agents** — Create and configure individual agents
- **Agent Teams** — Assemble agents into deployable teams
- **Prompt Library** — Central store for prompts and topics

### Using the Prompt Library

Central store for managing prompts and topics across agents. Use it to save
reusable prompts shared across agents, manage topics centrally for consistent
expertise boundaries, track prompt version lifecycle (authoring → testing →
production), and discover existing topics before creating duplicates.
Access via the **Prompt Library** tab in AI Agent Studio.

---

## Creating a Custom Agent — Step by Step

Follow this sequence: Tools → Agents → Agent Teams → Test → Publish → Deploy.

### Step 1: Create Tools

1. Navigate to **Tools > AI Agent Studio**
2. Click the **Tools** tab, then **Add**
3. Select the Tool Type:
   - **Document**: Upload documents for RAG. Set status to "Ready to publish"
   - **REST**: Define an HTTPS endpoint for external API calls
   - **Business Object**: Connect to Fusion business objects
4. Configure tool-specific settings and **Save**

When creating Document tools, upload relevant knowledge base files. Supported file
types: **PDF** (tagged or scanned), **TXT**, **DOCX**, **XLSX**, **PNG**, and **JPEG**.
Users can upload up to 5 files per interaction, with a total combined size limit
of 50 MB. For OCI-based RAG data sources, additionally supported: JSON, HTML, and
Markdown (MD), with a per-file limit of 100 MB.
The agent will use these for retrieval-augmented generation.

### Step 2: Create an Agent

1. Click the **Agents** tab, then **Add**
2. Fill in agent details:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Suchert/oracle-ai-agent-studio-skill](https://github.com/Suchert/oracle-ai-agent-studio-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
