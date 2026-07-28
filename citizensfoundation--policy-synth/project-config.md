---
trigger: always_on
description: The `PsAgent` class represents an AI agent instance in the PolicySynth Agents system. It is a Sequelize model mapped to the `ps_agents` table and encapsulates the configuration, associations, and management of an agent, including its relationships to users, groups, agent classes, connectors, and AI models.
---

# PsAgent

The `PsAgent` class represents an AI agent instance in the PolicySynth Agents system. It is a Sequelize model mapped to the `ps_agents` table and encapsulates the configuration, associations, and management of an agent, including its relationships to users, groups, agent classes, connectors, and AI models.

## Properties

| Name                | Type                                         | Description                                                                                 |
|---------------------|----------------------------------------------|---------------------------------------------------------------------------------------------|
| id                  | number                                       | Primary key. Unique identifier for the agent.                                               |
| uuid                | string                                       | Universally unique identifier for the agent.                                                |
| user_id             | number                                       | Foreign key referencing the user who owns/created the agent.                                |
| created_at          | Date                                         | Timestamp when the agent was created.                                                       |
| updated_at          | Date                                         | Timestamp when the agent was last updated.                                                  |
| class_id            | number                                       | Foreign key referencing the agent class (type/blueprint of the agent).                      |
| group_id            | number                                       | Foreign key referencing the group to which the agent belongs.                               |
| configuration       | PsBaseNodeConfiguration                      | JSON configuration object for the agent instance.                                           |
| parent_agent_id     | number \| undefined                          | Optional foreign key referencing the parent agent (for agent hierarchies).                  |
| Class               | PsAgentClass \| undefined                    | Associated agent class instance.                                                            |
| User                | YpUserData \| undefined                      | Associated user instance.                                                                   |
| Group               | YpGroupData \| undefined                     | Associated group instance.                                                                  |
| ExternalApiUsage    | PsExternalApiUsageAttributes[] \| undefined  | Associated external API usage records.                                                      |
| ModelUsage          | PsModelUsageAttributes[] \| undefined        | Associated AI model usage records.                                                          |
| ParentAgent         | PsAgent \| undefined                         | Associated parent agent instance.                                                           |
| SubAgents           | PsAgent[] \| undefined                       | Associated sub-agent instances (children).                                                  |
| AiModels            | PsAiModelAttributes[] \| undefined           | Associated AI models (many-to-many).                                                        |
| InputConnectors     | PsAgentConnectorAttributes[] \| undefined    | Associated input connectors (many-to-many).                                                 |
| OutputConnectors    | PsAgentConnectorAttributes[] \| undefined    | Associated output connectors (many-to-many).                                                |

### Virtual Properties

| Name             | Type    | Description                                                      |
|------------------|---------|------------------------------------------------------------------|
| redisMemoryKey   | string  | Redis key for storing agent memory (e.g., `ps:agent:memory:...`). |
| redisStatusKey   | string  | Redis key for storing agent status (e.g., `ps:agent:status:...`). |

## Methods

### Connector Association Methods

| Name                      | Parameters                                      | Return Type         | Description                                      |
|---------------------------|------------------------------------------------|---------------------|--------------------------------------------------|
| addInputConnector         | connector: PsAgentConnector, obj?: any         | Promise<void>       | Add a single input connector.                    |
| addInputConnectors        | connectors: PsAgentConnector[]                 | Promise<void>       | Add multiple input connectors.                   |
| getInputConnectors        |                                                | Promise<PsAgentConnector[]> | Get all input connectors.                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CitizensFoundation/policy-synth](https://github.com/CitizensFoundation/policy-synth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
