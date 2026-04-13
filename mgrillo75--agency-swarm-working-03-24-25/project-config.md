---
trigger: always_on
description: Agency Swarm is a framework that allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools and agents that fulfill specific needs within the agency. This involves:
---

# AI Agent Creator Instructions for Agency Swarm Framework

Agency Swarm is a framework that allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools and agents that fulfill specific needs within the agency. This involves:

1. **Planning**: First, plan step-by-step the Agency strcuture, which tools each agent must use and the best possible packages or APIs to create each tool based on the user's requirements. Ask the user for clarification before proceeding if you are unsure about anything.
2. **Folder Structure and Template Creation**: Create the Agent Templates for each agent using the CLI Commands provided below.
3. **Tool Development:** Develop each tool and place it in the correct agent's tools folder, ensuring it is robust and ready for production environments.
4. **Agent Creation**: Create agent classes and instructions for each agent, ensuring correct folder structure.
5. **Agency Creation**: Create the agency class in the agency folder, properly defining the communication flows between the agents.
6. **Testing**: Test each tool for the agency, and the agency itself, to ensure they are working as expected.
7. **Iteration**: Repeat the above steps as instructed by the user, until the agency performs consistently to the user's satisfaction.

You will find a detailed guide for each of the steps below.

# Step 1: Planning

Before proceeding with the task, make sure you have the following information:

- The mission and purpose of the agency.
- Description of the operating environment of the agency.
- The roles and capabilities of each agent in the agency.
- The tools each agent will use and the specific APIs or packages that will be used to create each tool.
- Communication flows between the agents.

If any of the above information is not provided, ask the user for clarification before proceeding.

# Step 2: Folder Structure and Template Creation

To create the folder structure and agent templates, follow these steps:

1. Create the main folder for the agency with the following command:

   ```bash
   mkdir <agency_name>
   ```

2. Create the Agent Templates inside the agency folder for each agent using the CLI command below:

   ```bash
   agency-swarm create-agent-template --name "AgentName" --description "Agent Description" --path "/path/to/agency/folder"
   ```

   You must repeat this step for each agent in the agency. Make sure to correctly specify the path to the agency folder.

### Understanding the Folder Structure

After creating the templates, the folder structure is organized as follows:

```
agency_name/
├── agent_name/
│   ├── __init__.py
│   ├── agent_name.py
│   ├── instructions.md
│   └── tools/
│       ├── tool_name1.py
│       ├── tool_name2.py
│       ├── tool_name3.py
│       ├── ...
├── another_agent/
│   ├── __init__.py
│   ├── another_agent.py
│   ├── instructions.md
│   └── tools/
│       ├── tool_name1.py
│       ├── tool_name2.py
│       ├── tool_name3.py
│       ├── ...
├── agency.py
├── agency_manifesto.md
├── requirements.txt
└──...
```

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

Follow this folder structure when further creating or modifying any files.

# Step 3: Tool Creation

Tools are the specific actions that agents can perform. They are defined using pydantic, which provides a convenient interface and automatic type validation.

#### 1. Import Necessary Modules

Start by importing `BaseTool` from `agency_swarm.tools` and `Field` from `pydantic`. These imports will serve as the foundation for your custom tool class. Import any additional packages necessary to implement the tool's logic based on the user's requirements. Import `load_dotenv` from `dotenv` to load the environment variables.

```python
from agency_swarm.tools import BaseTool
from pydantic import Field
import os
from dotenv import load_dotenv

load_dotenv() # always load the environment variables
```

#### 2. Define Your Tool Class and Docstring

Create a new class that inherits from `BaseTool`. Write a clear docstring describing the tool’s purpose. This docstring is crucial as it helps agents understand how to use the tool. `BaseTool` extends `BaseModel` from pydantic.

```python
class MyCustomTool(BaseTool):
    """
    A brief description of what the custom tool does.
    The docstring should clearly explain the tool's purpose and functionality.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mgrillo75) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
