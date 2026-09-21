---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Multi-Agent MCP Ensemble System** - a revolutionary self-expanding AI system that can dynamically create specialized agents and connect to external MCP (Model Context Protocol) servers from the internet. The system demonstrates advanced distributed AI architecture with recursive self-improvement capabilities.

## Core System Capabilities

### 🤖 Dynamic Agent Creation
The system can create new agents on-demand based on task requirements:
- Analyze task descriptions to infer required capabilities
- Generate complete agent code with best practices
- Deploy agents at runtime and integrate them into the system
- Support recursive agent creation (agents creating other agents)

### 🌐 External MCP Integration
Connect to 30+ external MCP servers across the internet:
- AI/ML services (OpenAI, Anthropic, Hugging Face)
- Development tools (GitHub, Docker Hub, AWS)
- Data services (Kaggle, Alpha Vantage, Airtable)
- Communication platforms (Slack, Discord, Notion)

### 🧠 Meta-Task Processing
Handle complex multi-step workflows:
- Decompose high-level tasks into executable subtasks
- Route tasks to appropriate agents (existing or newly created)
- Manage task dependencies and parallel execution
- Synthesize results from multiple agents

## Architecture

### Key Components

**Enhanced Coordinator** (`src/agents/coordinator-enhanced.js`)
- Orchestrates meta-tasks and agent creation
- Handles intelligent task decomposition
- Manages agent lifecycle and communication

**Agent Factory** (`src/agents/specialists/agent-factory.js`)
- Creates specialized agents based on requirements
- Generates agent code with capabilities and MCP connections
- Manages agent deployment and registration

**External MCP Registry** (`src/core/external-mcp-registry.js`)
- Maintains registry of 30+ external MCP servers
- Provides intelligent server discovery and selection
- Handles authentication and connection testing

**Base Agent Framework** (`src/core/base-agent.js`)
- Foundation for all agents with event-driven architecture
- MCP client management and tool invocation
- Health monitoring and error recovery

### Message Flow
```
User Task → Enhanced Coordinator → Task Decomposition → Agent Factory (if needed) 
→ MCP Connections → Task Execution → Result Synthesis → Response
```

## Development Guidelines

### Working with Meta-Tasks
When users request tasks that involve creating agents or connecting to external services:

1. **Recognize Meta-Task Patterns**:
   - Tasks mentioning "create agent", "connect to external service", "use AI/ML"
   - Tasks requiring capabilities not available in existing agents
   - Tasks involving multiple external platforms

2. **Task Decomposition Process**:
   - Analyze task requirements and infer needed capabilities
   - Determine if new agents are needed
   - Identify optimal external MCP servers
   - Create execution plan with dependencies

3. **Agent Creation**:
   - Use Agent Factory to generate specialized agents
   - Ensure agents include proper MCP server connections
   - Follow established patterns for capability implementation

### Common Development Tasks

#### Build and Run System
```bash
# Install dependencies
npm install

# Start complete ensemble
npm run start:all

# Start individual components
npm run start:coordinator
npm run start:code-agent
```

#### Submit Meta-Tasks
```bash
# Create data science agent with external connections
node src/index.js task "Create a data science agent that uses Kaggle for data and OpenAI for analysis"

# Create marketing agent with social media integration
node src/index.js task "Create a marketing agent that manages campaigns across Slack and Discord"
```

#### Add New Agent Types
1. Create agent class extending `BaseAgent`
2. Add configuration to `config/ensemble.yaml`
3. Update Agent Factory templates
4. Add MCP server recommendations

#### Integrate New MCP Servers
1. Add server config to `external-mcp-registry.js`
2. Implement authentication handling
3. Update agent recommendations
4. Test connections and error handling

### Testing
```bash
# Run all tests
npm test

# Test specific components
npm test -- --testNamePattern="BaseAgent"

# Run with coverage
npm test -- --coverage
```

### Logging and Monitoring
- Use structured logging with agent/task context
- Monitor agent health and MCP server connections
- Log all meta-task operations for debugging

## Key Files to Understand

### Core Framework
- `src/core/base-agent.js` - Base agent implementation
- `src/core/message-bus.js` - Redis communication system
- `src/core/mcp-manager.js` - MCP server management
- `src/core/external-mcp-registry.js` - External server registry

### Agent System
- `src/agents/coordinator-enhanced.js` - Meta-task coordinator
- `src/agents/specialists/agent-factory.js` - Dynamic agent creator
- `src/agents/specialists/code-agent.js` - Programming agent
- `src/agents/specialists/research-agent.js` - Research agent

### System Management
- `src/ensemble-launcher.js` - System orchestration
- `src/index.js` - CLI interface
- `config/ensemble.yaml` - System configuration

### Documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ncg87/a2a-mcp](https://github.com/ncg87/a2a-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
