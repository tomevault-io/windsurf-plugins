---
trigger: always_on
description: Structure agent code for Azure's `azd ai` command. Use when users mention "azd ai", "azd init agent", "Foundry agent", "scaffold agent", "convert to azd", "update for azd", "upgrade to azd ai", "fix azd ai", "migrate to Foundry", or want to deploy, convert, update, fix, or upgrade an AI agent for Azure.
---


# Azure AI Agent Scaffolding Skill

This skill helps developers prepare their AI agent code for deployment to Azure AI Foundry using the `azd ai` extension of the Azure Developer CLI.

## When to Use This Skill

Use this skill when a user wants to:
- **Scaffold a new agent from scratch** (greenfield project with no existing code)
- Convert existing agent code to the `azd ai` expected format
- Scaffold a new Azure AI Foundry agent project from scratch
- Structure their agent for deployment with `azd up`
- Understand what files and configuration `azd ai` requires
- Migrate from other agent frameworks to Azure AI Foundry hosted agents

## Core Workflow

### Step 1: Analyze the User's Current Project

First, understand what the user has:

1. **Detect existing code**: Look for agent implementations (Python, TypeScript, etc.)
2. **Identify the agent framework**: LangGraph, Semantic Kernel, AutoGen, custom, etc.
3. **Find entry points**: main.py, index.ts, or other entry files
4. **Check for existing configuration**: azure.yaml, agent.yaml, Dockerfile, requirements.txt, package.json

### Step 2: Generate Required Files

The `azd ai` extension expects a specific project structure:

```
project-root/
├── azure.yaml              # Project configuration (REQUIRED)
├── infra/                  # Bicep infrastructure files (REQUIRED)
│   ├── main.bicep
│   ├── main.parameters.json
│   └── core/               # Reusable Bicep modules
│       └── ai/
│           └── ai-project.bicep
└── src/
    └── <AgentName>/        # Agent source folder (REQUIRED)
        ├── agent.yaml      # Agent definition (REQUIRED)
        ├── Dockerfile      # Container build file (REQUIRED)
        ├── main.py         # Agent entry point
        └── requirements.txt
```

### Step 3: Create Configuration Files

#### azure.yaml (Project Root)

This is the main project configuration file that defines services and infrastructure:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/Azure/azure-dev/main/schemas/v1.0/azure.yaml.json

requiredVersions:
    extensions:
        azure.ai.agents: '>=0.1.0-preview'

name: <project-name>

services:
    <AgentName>:
        project: src/<AgentName>
        host: azure.ai.agent
        language: docker
        docker:
            remoteBuild: true
        config:
            container:
                resources:
                    cpu: "1"
                    memory: 2Gi
                scale:
                    maxReplicas: 3
                    minReplicas: 1
            deployments:
                - model:
                    format: OpenAI
                    name: gpt-4o-mini
                    version: "2024-07-18"
                  name: gpt-4o-mini
                  sku:
                    capacity: 10
                    name: GlobalStandard

infra:
    provider: bicep
    path: ./infra
```

#### agent.yaml (Inside src/<AgentName>/)

Defines the agent's metadata, protocols, and environment variables:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/microsoft/AgentSchema/refs/heads/main/schemas/v1.0/ContainerAgent.yaml

kind: hosted
name: <AgentName>
description: "<Brief description of what the agent does>"

metadata:
    authors:
        - <author-name>
    example:
        - content: "<Example user prompt - always quote strings with special characters>"
          role: user
    tags:
        - <tag1>
        - <tag2>

protocols:
    - protocol: responses
      version: v1

environment_variables:
  - name: FOUNDRY_PROJECT_ENDPOINT
    value: ${AZURE_AI_PROJECT_ENDPOINT}
  - name: FOUNDRY_MODEL_DEPLOYMENT_NAME
    value: gpt-4o-mini
  - name: APPLICATIONINSIGHTS_CONNECTION_STRING
    value: ${APPLICATIONINSIGHTS_CONNECTION_STRING}
```

**Note:** Set `FOUNDRY_MODEL_DEPLOYMENT_NAME` to match the deployment name in your `azure.yaml` (e.g., `gpt-4o-mini`).

**⚠️ Environment Variable Naming:** The hosted agent platform injects variables with `FOUNDRY_` prefix. Your Python code must read `FOUNDRY_PROJECT_ENDPOINT` and `FOUNDRY_MODEL_DEPLOYMENT_NAME` (not `AZURE_*` prefixes). The `agent.yaml` maps Azure outputs to the expected names.

**IMPORTANT YAML Formatting Rules:**
- Always wrap `content:` and `description:` values in double quotes
- Escape internal quotes with backslash: `"He said \"hello\""`
- Strings with colons, commas, or special characters MUST be quoted

#### Dockerfile

Standard Python container for hosted agents:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY ./ user_agent/

WORKDIR /app/user_agent

RUN if [ -f requirements.txt ]; then \
        pip install -r requirements.txt; \
    else \
        echo "No requirements.txt found"; \
    fi

EXPOSE 8088

ENV PORT=8088

CMD ["python", "main.py"]
```

For TypeScript/Node.js agents:

```dockerfile
FROM node:20-slim

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 8088

ENV PORT=8088

CMD ["node", "dist/main.js"]
```

### Step 4: Adapt the Agent Code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spboyer/skill-azd-ai-init](https://github.com/spboyer/skill-azd-ai-init) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
