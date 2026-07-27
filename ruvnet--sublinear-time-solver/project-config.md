---
trigger: always_on
description: The **Psycho-Symbolic Reasoner** is an advanced AI agent that combines symbolic reasoning with psychological context understanding to perform sophisticated analysis and knowledge processing. This agent leverages structured knowledge graphs and psychological insights to provide nuanced reasoning capabilities for complex problem-solving scenarios.
---

# Psycho-Symbolic Reasoner Agent

## Overview

The **Psycho-Symbolic Reasoner** is an advanced AI agent that combines symbolic reasoning with psychological context understanding to perform sophisticated analysis and knowledge processing. This agent leverages structured knowledge graphs and psychological insights to provide nuanced reasoning capabilities for complex problem-solving scenarios.

## Agent Configuration

```yaml
name: "Psycho-Symbolic Reasoner"
description: "AI agent that combines symbolic reasoning with psychological context understanding"
version: "1.0.0"
capabilities:
  - symbolic_reasoning
  - psychological_analysis
  - knowledge_graph_processing
  - reasoning_chain_analysis
  - contextual_understanding
```

## MCP Server Configuration

To use this agent, you must first add the psycho-symbolic-reasoner MCP server to your Claude configuration:

```bash
# Add the MCP server
claude mcp add psycho-symbolic-reasoner npx psycho-symbolic-reasoner mcp start

# Verify the server is running
claude mcp list
```

### Server Details
- **Server Name**: `psycho-symbolic-reasoner`
- **Protocol**: MCP (Model Context Protocol)
- **Transport**: stdio
- **Capabilities**: Reasoning analysis, knowledge graph operations, psychological context processing

## Available Tools

### 1. `mcp__psycho-symbolic-reasoner__reason`
Performs comprehensive psycho-symbolic reasoning analysis on provided inputs.

**Parameters:**
- `query` (string, required): The reasoning query or problem to analyze
- `context` (object, optional): Additional context for reasoning
- `reasoning_type` (enum, optional): Type of reasoning to perform
  - `deductive`: Logical deduction from premises
  - `inductive`: Pattern recognition and generalization
  - `abductive`: Best explanation inference
  - `analogical`: Reasoning by analogy
  - `causal`: Cause-and-effect reasoning
- `psychological_factors` (array, optional): Psychological aspects to consider
- `confidence_threshold` (number, optional): Minimum confidence level (0-1)

**Usage Example:**
```typescript
await mcp__psycho_symbolic_reasoner__reason({
  query: "What are the implications of cognitive bias in decision-making processes?",
  reasoning_type: "causal",
  psychological_factors: ["cognitive_bias", "decision_making", "heuristics"],
  confidence_threshold: 0.8
});
```

### 2. `mcp__psycho-symbolic-reasoner__knowledge_graph_query`
Queries the internal knowledge graph for relevant information and relationships.

**Parameters:**
- `query` (string, required): Query to search in the knowledge graph
- `entity_types` (array, optional): Specific entity types to filter
- `relationship_types` (array, optional): Relationship types to include
- `depth` (number, optional): Maximum traversal depth (default: 3)
- `limit` (number, optional): Maximum results to return (default: 50)

**Usage Example:**
```typescript
await mcp__psycho_symbolic_reasoner__knowledge_graph_query({
  query: "psychological theories related to reasoning",
  entity_types: ["theory", "concept", "researcher"],
  relationship_types: ["relates_to", "developed_by", "applies_to"],
  depth: 2,
  limit: 20
});
```

### 3. `mcp__psycho-symbolic-reasoner__add_knowledge`
Adds new facts, relationships, or entities to the knowledge base.

**Parameters:**
- `knowledge_type` (enum, required): Type of knowledge to add
  - `fact`: Simple factual statement
  - `relationship`: Connection between entities
  - `rule`: Logical rule or principle
  - `concept`: New concept definition
- `content` (object, required): Knowledge content structure
- `source` (string, optional): Source of the knowledge
- `confidence` (number, optional): Confidence in the knowledge (0-1)
- `tags` (array, optional): Tags for categorization

**Usage Example:**
```typescript
await mcp__psycho_symbolic_reasoner__add_knowledge({
  knowledge_type: "relationship",
  content: {
    subject: "cognitive_load_theory",
    predicate: "influences",
    object: "learning_effectiveness",
    properties: {
      strength: "strong",
      context: "educational_psychology"
    }
  },
  source: "Sweller, J. (1988)",
  confidence: 0.9,
  tags: ["psychology", "learning", "cognition"]
});
```

### 4. `mcp__psycho-symbolic-reasoner__analyze_reasoning_path`
Analyzes the reasoning chain and provides insights into the reasoning process.

**Parameters:**
- `reasoning_id` (string, required): ID of the reasoning session to analyze
- `analysis_type` (enum, optional): Type of analysis to perform
  - `logical_validity`: Check logical soundness
  - `psychological_plausibility`: Assess psychological realism
  - `bias_detection`: Identify potential biases
  - `confidence_calibration`: Evaluate confidence levels
- `include_suggestions` (boolean, optional): Include improvement suggestions
- `detailed_breakdown` (boolean, optional): Provide step-by-step analysis

**Usage Example:**
```typescript
await mcp__psycho_symbolic_reasoner__analyze_reasoning_path({
  reasoning_id: "reasoning_session_12345",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruvnet/sublinear-time-solver](https://github.com/ruvnet/sublinear-time-solver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
