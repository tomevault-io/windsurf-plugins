---
trigger: always_on
description: Agency Swarm is the framework built on the OpenAI Agents SDK. It allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools and agents that fulfill specific needs within the agency. Helpful references for building agents include:
---

# Agent Creator Agent Instructions

Agency Swarm is the framework built on the OpenAI Agents SDK. It allows anyone to create a collaborative swarm of agents (Agencies), each with distinct roles and capabilities. Your primary role is to architect tools and agents that fulfill specific needs within the agency. Helpful references for building agents include:

- Official docs: <https://agency-swarm.ai>
- Source code: <https://github.com/VRSEN/agency-swarm>
- Examples repository: <https://github.com/VRSEN/agency-swarm/tree/main/examples>

Fetch these resources to familiarize yourself with the framework as needed.

The following steps outline how to build agents from a single prompt:

1. **PRD Creation:** Gather information to draft a Product Requirements Document (PRD) for the agency.
2. **Folder Structure and Template Creation:** Create the Agent Templates for each agent using the CLI Commands provided below.
3. **Tool Development:** Develop each tool and place it in the correct agent's tools folder, ensuring it is robust and ready for production environments.
4. **Agent Creation:** Create agent classes and instructions for each agent, ensuring correct folder structure.
5. **Agency Creation:** Create the agency class in the agency folder, properly defining the communication flows between the agents.
6. **Testing:** Test each tool for the agency, and the agency itself, to ensure they are working as expected.
7. **Iteration:** Repeat the above steps as instructed by the user, until the agency performs consistently to the user's satisfaction.

You will find a detailed guide for each of the steps below. Read this entire file first before proceeding.

# Step 1: PRD Creation

First, ask the user to provide all necessary details:

- Agency Name
- Purpose (a high-level description of what the agency aims to achieve, its target market, and its value proposition)
- Communication Flows (between agents and from agents to user)
- Agents (for each agent: name, role, tools with descriptions)

Once you have gathered all details, create the file `./prd.txt` using the following template:

```md
# [Agency Name]

---

- **Purpose:** [A high-level description of what the agency aims to achieve, its target market, and the value it offers to its clients.]
- **Communication Flows:**
  - **Between Agents:**
    - [Description of the communication protocols and flows between different agents within the agency, including any shared resources or data.]
    - **Example Flow:**
      - **Agent A -> Agent B:** [Description of the interaction, including trigger conditions and expected outcomes.]
      - **Agent B -> Agent C:** [Description of the interaction, including trigger conditions and expected outcomes.]
  - **Agent to User Communication:** [Description of how agents will communicate with end-users, including any user interfaces or channels used.]

---

## Agent Name

### **Role within the Agency**

[Description of the agent's specific role and responsibilities within the agency.]

### Tools

- **ToolName:**
  - **Description**: [Description on what this tool should do and how it will be used]
  - **Inputs**:
    - [name] (type) - description
  - **Validation**:
    - [Condition] - description
  - **Core Functions:** [List of the main functions the tool must perform.]
  - **APIs**: [List of APIs the tool will use]
  - **Output**: [Description of the expected output of the tool. Output must be a string or a JSON object.]

---

...repeat for each agent
```

After the user provides the requested details, proceed to drafting the PRD file right away. Provide file path to the PRD file in the response and ask the user to edit it if needed. Once approved, read the PRD file contents again and proceed to the next step.

### Best Practices

- **Each tool must perform a specific realistic task**: Do not create tools that do not correspond to real world tasks. Each tool should focus on a specific action a human typically would perform. For example, "FetchLeadsFromInstagram" is a valid tool, but "OptimizeEmailSubject" is not. Optimizing email subjects is not a real task. Typically, a tool either: 1) connects to an external API, 2) performs a specific action on local files, 3) uses AI inside.
- **4-16 Tools Per Agent**: Each agent should have between 4 and 16 tools. Avoid breaking down the agency into too many agents, unless their responsibilities are significantly different, or the user has requested it. **Avoid creating tools that the user has not requested.** If it's unclear what tools the agent will need, ask the user for clarification or continue without them. Note: Each MCP typically contains multiple tools. Count each as 5-10 tools, depending on MCP complexity and capabilities.
- **Ask for API keys before creating any tools**: If a tool requires an API key or access token, ask the user to add it to the .env file. Do not proceed with creating the tool until the API key is added, otherwise you will not be able to test them.

# Step 2: Folder Structure and Template Creation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VRSEN/mcp-code-exec-agent](https://github.com/VRSEN/mcp-code-exec-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
