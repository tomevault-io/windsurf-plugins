---
trigger: always_on
description: AI Agent Creator Instructions for Agency Swarm Framework
---


Agency Swarm is the framework built on the OpenAI Agents SDK. It allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools and agents that fulfill specific needs within the agency.

The following steps outline how to build AI agents with Agency Swarm:

0. **Setup**: Create a to-do list for yourself and activate a virtual environment. If virtual environment does not exist, create it.
1. **Project Exploration:** Understand the existing project structure, check for PRD, and remove example agents if present.
2. **Folder Structure and Template Creation:** Create the Agent Templates for each agent using the CLI Commands provided below.
3. **Tool Development:** Develop each tool and place it in the correct agent's tools folder, ensuring it is robust and ready for production environments.
4. **Agent Creation:** Create agent classes and instructions for each agent, ensuring correct folder structure.
5. **Agency Creation:** Create the agency class in the agency folder, properly defining the communication flows between the agents.
6. **Testing:** Test each tool for the agency, and the agency itself, to ensure they are working as expected.
7. **Iteration:** Repeat the above steps as instructed by the user, until the agency performs consistently to the user's satisfaction.

You will find a detailed guide for each of the steps below. Read this entire file first before proceeding.

# Step 1: Project Exploration

Before starting any work, you must understand the current state of the project and prepare it for agent creation.

## Exploration Checklist

1. **Check root directory structure**: List files and folders in the project root
2. **Look for PRD**: Check if `prd.txt` exists in the root directory
3. **Check for example agents**: Look for folders like `example_agent/` or `example_agent2/`
4. **Review existing files**: Read `agency.py`, `shared_instructions.md`, `agent_name/instructions.md`, `agent_name/tools/`, etc.

## Actions Required

### If PRD exists:

- Read the entire `prd.txt` file to understand the agency structure
- Verify agent roles are realistic (modeled after real job positions)
- Confirm tool specifications are clear and actionable
- Proceed to Step 2 (Folder Structure)

### If PRD does not exist:

- **Proceed directly with user instructions**

### If example agents exist:

- **Remove all example agent folders**: Delete `example_agent/` and `example_agent2/` completely
- **Clean up agency.py**: Remove example agent imports and communication flows

### If the task required connecting to external systems (slack, notion, etc):

- Check if the user has added the API keys to the .env file.
- If not, ask the user to add the API keys before proceeding with the task.
- Do not proceed with the task until the API keys are added, otherwise you will not be able to test them.

# Step 2: Folder Structure and Template Creation

After exploration, setup the folder structure for each agent.

The basic folder structure is already created for you:

```
├── example_agent/
│   ├── __init__.py
│   ├── example_agent.py
│   ├── instructions.md
│   ├── files/
│   └── tools/
│       ├── ToolName.py
│       ├── ToolName2.py
│       ├── ToolName3.py
│       ├── ...
├── example_agent2/
│   ├── __init__.py
│   ├── example_agent2.py
│   ├── instructions.md
│   ├── files/
│   └── tools/
│       ├── ToolName.py
│       ├── ToolName2.py
│       ├── ToolName3.py
│       ├── ...
├── agency.py
├── shared_instructions.md
├── requirements.txt
├── .env
└──...
```

**Rules for the folder structure:**

- Agency folder must be named in lowercase, with underscores instead of spaces.
- Each agency and agent has its own dedicated folder.
- Within each agent folder:

  - A 'tools' folder contains all tools for that agent.
  - An 'instructions.md' file provides agent-specific instructions.
  - An '**init**.py' file contains the import of the agent.

- Tool Import Process:

  - Create a file in the 'tools' folder with the same name as the tool class.
  - Tools are automatically imported to the agent class.
  - All new requirements must be added to the requirements.txt file.

- Agency Configuration:
  - The 'agency.py' file is the main file where all new agents are imported.
  - When creating a new agency folder, use descriptive names, like for example: marketing_agency, development_agency, etc.
  - Create a `.env` file in the root folder and add a placeholder for `OPENAI_API_KEY` and any other API keys that are required by the tools for the user to fill in.

Follow this folder structure when further creating or modifying any files. Replace example_agent folders with the actual agents when creating agents for the first time.

**To create a new agent template, use the following command:**

```bash
agency-swarm create-agent-template --description "Description of the agent" --model "gpt-5" --reasoning "medium" "agent_name"
```

### Best Practices for Agent Structures

- **Realistic Agent Roles**: Model agents after actual job positions, not task-specific roles
  - ✅ Good: "Data Analyst", "Campaign Manager", "Financial Advisor"
  - ❌ Bad: "Chart Creator", "Email Sender", "Report Generator"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VRSEN) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
