---
trigger: always_on
description: Agency Swarm is a framework that allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools that fulfill specific needs within the agency. This involves:
---

# MCP Tool Creator Agent Instructions

Agency Swarm is a framework that allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools that fulfill specific needs within the agency. This involves:

0. **To-Do List Creation**: Create a to-do list of the steps to follow.
1. **Requirements Gathering**: Gather information to draft a Product Requirements Document (PRD) for the agency.
2. **Research & PRD Creation**: Search the web for the most relevant documentation about the packages and APIs that will be used to create the tools. Then, create the PRD file.
3. **Environment Setup**: Setup the environment for the tools.
4. **Tool Development**: Develop each tool and place it in the correct agent's tools folder, ensuring it is robust and ready for production environments.
5. **Testing**: Test each tool for the agency, and the agency itself, to ensure they are working as expected.

You will find a detailed guide for each of the steps below.

### Repository Structure

This is a repository that deploys tools as an MCP server. It has the following structure:

```
mcp-tools-template/
├── .cursor/
│   └── rules/
│       └── workflow.mdc  # Do not touch
│       └── PRD.md        # Product Requirements Document
├── tools/
│   ├── ToolName.py       # Shared tools go here (available to all MCP instances)
│   ├── marketing_mcp/    # MCP instance-specific tools (by convention use _mcp suffix)
│   │   └── ToolName.py
│   └── analytics_mcp/    # Another MCP instance
│       └── ToolName.py
├── server/
│   └── ...               # MCP server files (do not touch)
├── requirements.txt      # Dependencies
├── .env                  # Environment variables
├── README.md
└── [other files]         # Do not touch
```

**Folder Structure Rules**:

Follow this folder structure when further creating or modifying any files.

  - The main 'tools' folder contains shared tools available to all MCP instances.
  - By convention, create subdirectories with "_mcp" suffix for separate MCP instances (e.g., marketing_mcp, analytics_mcp).
  - Tool files must be named exactly as the tool class name, with the .py extension.
  - Tools in these folders are automatically deployed to the MCP server.
  - All new tool requirements must be added to the requirements.txt file.

# Step 0: To-Do List Creation

Before starting the workflow, create a to-do list following all the steps below. 

**Notes**: 
- Step 4 should be split into multiple to-do items. Each tool should be in a separate to-do item.
- Iteration should not be in the to-do list. Instead, create a new to-do list for each iteration from scratch.

# Step 1: Requirements Gathering

First, ask the user to provide all necessary details:
- Purpose (a high-level description of what the agency aims to achieve, its target market, and its value proposition)
- Tools (for each tool: name, description, inputs, outputs, validation, core functions, APIs)

Ask any clarifying questions to the user. For example, "What inputs should the tool take?", "How is the agent going to use the tool together with other tools?", "What outputs should the tool return?", etc.

# Step 2: Research & PRD Creation

Once you have gathered all details, search the web for the most relevant documentation about the packages and APIs that will be used to create the tools. Then, create the file `.cursor/rules/prd.md` using the following template:

```md
# [Agency Name]

---

- **ToolName:**
    - **Description**: [Description on what this tool should do and how it will be used]
    - **Inputs**:
        - [name] (type) - description
    - **Validation**:
        - [Condition] - description
    - **Core Functions:** [List of the main functions the tool must perform.]
    - **APIs**: [List of APIs the tool will use]
    - **Output**: [Description of the expected output of the tool. Output must be a string or a JSON object.]

#...repeat for each tool...
```

After the user provides the requested details, proceed to drafting the PRD file right away. Provide file path to the PRD file in the response and ask the user to edit it if needed. Once approved, read the PRD file content changes and proceed to the next step.

# Step 3: Environment Setup

Ask the user to provide all the necessary environment variables for the tools in the `./.env` file. You do not have access to this file, so do not try to read it. Simply output all the environment variable names that the user needs to add like `OPENAI_API_KEY`, `SLACK_BOT_TOKEN`, etc. in chat. Once the user has saved all the environment variables, make sure the python virtualenvironment is setup and activated:

1. Check if the python virtual environment is activated:
    ```bash
    which python
    ```
    ❗If this outputs global python, you need to create and activate a virtual environment.
2. If the python virtual environment is not activated, create and activate it using the following command:

    ```bash
    python -m venv venv && source venv/bin/activate
    ```

Do not run any commands globally.

# Step 4: Tool Development


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eybersjp/mcp-tools-deployment-template](https://github.com/eybersjp/mcp-tools-deployment-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
