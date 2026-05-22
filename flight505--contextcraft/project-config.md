---
trigger: always_on
description: You Must use the MCP Think Tank to extend your AI assistant with structured reasoning, persistent memory, task management, and web research capabilities.
---

You Must use the MCP Think Tank to extend your AI assistant with structured reasoning, persistent memory, task management, and web research capabilities. 

### 1. Thinking Tools
- `think` - Use for complex reasoning, architecture planning, and step-by-step problem-solving
  - **When to use**: For important decisions requiring structured analysis or reflection
  - **Pro tip**: Add `storeInMemory: true` to save important thoughts in the knowledge graph
  - **Example**: `think({structuredReasoning: "Analyzing pros/cons of database options...", storeInMemory: true})`

### 2. Memory & Knowledge Graph
- `create_entities` - Add new concepts to the knowledge graph
  - **When to use**: To document important concepts, systems, or components
  - **Example**: `create_entities([{name: "AuthSystem", entityType: "System", observations: ["Uses JWT"]}])`

- `add_observations` - Add facts to existing entities
  - **When to use**: When new information about an existing entity is discovered
  - **Example**: `add_observations([{entityName: "AuthSystem", contents: ["Expires tokens after 24h"]}])`

- `create_relations` - Create relationships between entities
  - **When to use**: To connect related concepts in the knowledge graph
  - **Example**: `create_relations([{from: "AuthSystem", to: "Security", relationType: "enhances"}])`

- `read_graph` - Get the entire knowledge graph
  - **When to use**: To understand the full project context before making major decisions

- `search_nodes` - Find entities matching search criteria
  - **When to use**: To retrieve relevant knowledge before solving a problem
  - **Example**: `search_nodes({query: "authentication"})`

- `open_nodes` - Retrieve specific entities by name
  - **When to use**: When you need details about a specific known entity
  - **Example**: `open_nodes({names: ["AuthSystem"]})`

- `update_entities` - Modify existing entities
  - **When to use**: When core information about an entity changes
  - **Example**: `update_entities([{name: "AuthSystem", entityType: "Security"}])`

- `update_relations` - Modify relationships between entities
  - **When to use**: When the nature of a relationship changes
  - **Example**: `update_relations([{from: "AuthSystem", to: "Security", relationType: "implements"}])`

- `delete_entities` - Remove entities from the knowledge graph
  - **When to use**: When concepts become obsolete or are merged
  - **Example**: `delete_entities({entityNames: ["OldSystem"]})`

- `delete_observations` - Remove specific facts from entities
  - **When to use**: When information becomes outdated or incorrect
  - **Example**: `delete_observations([{entityName: "AuthSystem", observations: ["Uses MD5"]}])`

- `delete_relations` - Remove relationships between entities
  - **When to use**: When connections are no longer valid
  - **Example**: `delete_relations([{from: "AuthSystem", to: "Legacy", relationType: "depends on"}])`

### 3. Task Management
- `plan_tasks` - Create and organize project tasks
  - **When to use**: At project start or when planning new features
  - **Example**: `plan_tasks([{description: "Implement login", priority: "high"}])`

- `list_tasks` - Retrieve tasks filtered by status/priority
  - **When to use**: To understand current work status
  - **Example**: `list_tasks({status: "todo"})`

- `next_task` - Get and start the highest priority task
  - **When to use**: When ready to work on the next priority
  - **Example**: `next_task({random_string: ""})`

- `complete_task` - Mark tasks as done
  - **When to use**: When a task is finished
  - **Example**: `complete_task({id: "task-uuid"})`

- `update_tasks` - Modify existing tasks
  - **When to use**: When priorities change or task details need updating
  - **Example**: `update_tasks([{id: "task-uuid", priority: "high"}])`

- `show_memory_path` - Get the location of the knowledge graph file
  - **When to use**: For debugging or when examining stored knowledge directly
  - **Example**: `show_memory_path({random_string: ""})`

### 4. When user mentions "Research"
When the user mentions research, use the following four tools. 
- `exa_search` - Search the web for information
  - **When to use**: When current context is insufficient and you need external facts
  - **Example**: `exa_search({query: "latest React state management"})`

- `exa_answer` - Get sourced answers to specific questions
  - **When to use**: For factual questions requiring cited sources
  - **Example**: `exa_answer({question: "What is quantum computing?"})`

- `resolve-library-id` -  to retrieve a valid Context7-compatible library ID.
Parameters: libraryName: Library name to search for and retrieve a Context-compatible library ID.

- `get-library-docs` - Fetches up-to-date documentation for a library. You must call 'resolve-library-id' first to obtain the
exact Context/-compatible library ID required to use this tool.
Parameters:
• context7CompatibleLibraryID: Exact Context7-compatible library ID (e.g., 'mongodb/docs',
'vercel/nextjs') retrieved from 'resolve-library-id'.
• topic: Topic to focus documentation on (e.g., 'hooks', 'routing").

## Best Practices

### Thinking Process

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flight505/ContextCraft](https://github.com/flight505/ContextCraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
