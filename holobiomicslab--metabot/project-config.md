---
trigger: always_on
description: This document details the agent system in the application, including the specialized agents and their roles in processing queries.
---

# Agents API Reference 🤖

This document details the agent system in the application, including the specialized agents and their roles in processing queries.

## Common Architecture

All agents in the system share a similar architectural pattern and are managed by the agent factory ([`app.core.agents.agents_factory`](https://github.com/HolobiomicsLab/MetaboT/blob/main/app/core/agents/agents_factory.py)):

**Core Components:**

- **Dynamic Agent Creation**: Loads agent modules based on configuration settings
- **Flexible Parameter Handling**: Uses introspection to pass only required parameters to each agent
- **LLM Selection**: Supports configuration-based and agent-specific LLM selection
- **Session Management**: Maintains consistent session IDs across the agent ecosystem
- **Error Handling**: Provides robust logging and exception handling

### Core Factory Function

The `create_all_agents` function serves as the entry point for initializing the entire agent ecosystem:

```python
def create_all_agents(llms, graph, openai_key=None, session_id=None):
    """
    Dynamically create and initialize all agent modules as specified in the configuration.
    
    Parameters:
        llms (dict): A dictionary mapping LLM keys to their instances.
        graph: The graph instance used by the agents.
        openai_key (str, optional): The OpenAI API key to be used by agents.
        session_id (str, optional): A unique session identifier.
        
    Returns:
        dict: A dictionary mapping agent names to their created executor instances.
    """
```
## Individual Agent Architecture

Each agent in the system follows a consistent structural pattern while maintaining specialized functionality.

### Common Agent Components

- **Creation Function**: Each agent implements a `create_agent` function that returns an `AgentExecutor`
- **Tool Management**: Dynamically loads tools from its directory using the `import_tools` utility
- **Role-Specific Prompts**: Defines behavior through customized prompts
- **LLM Integration**: Utilizes language models as reasoning engines
- **Logging**: Implements consistent logging for monitoring and debugging

### Standard Agent Structure

```python
def create_agent(llms, graph, openai_key, llm_instance=None) -> AgentExecutor:
    """
    Creates and configures an agent with its specialized tools.
    
    Parameters:
        llms (dict): Available language models.
        graph: The knowledge graph instance.
        openai_key (str): API key for OpenAI services.
        llm_instance: Optional specific LLM instance to use.
        
    Returns:
        AgentExecutor: A configured agent executor instance.
    """
    # Load tools dynamically from the agent's directory
    # Configure the agent with appropriate prompts
    # Return an AgentExecutor instance
```
    
**Agent Locations:**

- **ENPKG Agent**: [`app/core/agents/enpkg/agent.py`](https://github.com/holobiomicslab/MetaboT/blob/main/app/core/agents/enpkg/agent.py)
- **Entry Agent**: [`app/core/agents/entry/agent.py`](https://github.com/HolobiomicsLab/MetaboT/blob/main/app/core/agents/entry/agent.py)
- **Interpreter Agent**: [`app/core/agents/interpreter/agent.py`](https://github.com/holobiomicslab/MetaboT/blob/main/app/core/agents/interpreter/agent.py)
- **SPARQL Agent**: [`app/core/agents/sparql/agent.py`](https://github.com/holobiomicslab/MetaboT/blob/main/app/core/agents/sparql/agent.py)
- **Validator Agent**: [`app/core/agents/validator/agent.py`](https://github.com/holobiomicslab/MetaboT/blob/main/app/core/agents/validator/agent.py)
- **Supervisor Agent**: [`app/core/agents/supervisor/agent.py`](https://github.com/holobiomicslab/MetaboT/blob/main/app/core/agents/supervisor/agent.py)

---

## Entry Agent 🚪

The Entry Agent serves as the first point of contact for user interactions.

**Purpose:**

- Initial query processing and classification
- File analysis for submitted documents


**Key Features:**

- Classifies queries into "New Knowledge Question" or "Help me understand Question"
- Analyzes submitted files using the FILE_ANALYZER tool
- Validates query completeness and requests clarification when needed

**Tools:**

- FILE_ANALYZER: Processes and analyzes submitted files, providing file paths and content summaries

**Usage Cases:**

- When users submit new queries requiring database information
- When files need to be analyzed
- For follow-up questions requiring context from previous conversations

---

## Validator Agent ✅

The Validator Agent ensures query validity and data quality.

**Purpose:**

- Validates user queries against database capabilities
- Ensures data quality and consistency
- Provides feedback for invalid queries

**Validation Checks:**

- Plant name verification in database
- Query compatibility with schema
- Content relevance to available nodes/entities

**Tools:**

- PLANT_DATABASE_CHECKER: Verifies plant names in database

**Validation Criteria:**

- Plant-specific and feature-related queries
- Grouping, counting, and annotation comparisons
- Schema compatibility
- Data availability

---

## Supervisor Agent 👨‍💼

The Supervisor Agent orchestrates the interaction between all other agents in the system.

**Purpose:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HolobiomicsLab/MetaboT](https://github.com/HolobiomicsLab/MetaboT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
