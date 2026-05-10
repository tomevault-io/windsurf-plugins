---
trigger: always_on
description: A high-level overview of how to extend Chat USD with custom agents, using the Navigation Agent as a reference implementation.
---


# Extending Chat USD with Custom Agents

This guide explains how to extend Chat USD with custom agents, using the Navigation Agent (`@omni.ai.langchain.agent.navigation`) as a reference implementation.

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Component Relationships](#component-relationships)
4. [Creating a Custom Agent](#creating-a-custom-agent)
   - [Extension Structure](#extension-structure)
   - [Node Implementation](#node-implementation)
   - [Modifier Implementation](#modifier-implementation)
   - [System Messages](#system-messages)
5. [Integration with Chat USD](#integration-with-chat-usd)
6. [Example: Navigation Agent](#example-navigation-agent)
7. [Best Practices](#best-practices)

## Introduction

Chat USD is a powerful AI assistant for Universal Scene Description (USD) development that can be extended with custom agents to add new capabilities. Custom agents allow Chat USD to perform specialized tasks, such as scene navigation, asset search, or custom operations on USD scenes.

This document explains how to create and integrate custom agents into the Chat USD framework, using the Navigation Agent as a reference implementation.

## Architecture Overview

Chat USD uses a modular architecture based on the Language Chain (LC) Agent framework. The key components for extending Chat USD with custom agents are:

1. **Extension**: Registers the agent with the system and manages its lifecycle.
2. **Nodes**: Implement the agent's functionality and define how it processes inputs and generates outputs.
3. **Modifiers**: Intercept and modify the behavior of nodes, allowing for custom processing of commands.
4. **System Messages**: Define the agent's capabilities, identity, and how it should respond to user queries.

## Component Relationships

Understanding the inheritance hierarchy and relationships between components is crucial for implementing a custom agent:

### Inheritance Hierarchy

1. **ChatUSDNavigationNetworkNode**: The main entry point for user interactions.
   - Derives from `ChatUSDNetworkNode` and `MultiAgentNetworkNode`, which handles routing between agents
   - Ultimately derives from `NetworkNode`, which is a container for subnodes
   - Does not interact with LLMs directly
   - Acts as a coordinator for the conversation between agents and supervisor
   - Registered with the user-friendly name "ChatUSD with navigation"

2. **ChatUSDNavigationSupervisorNode**: The orchestrator for agent interactions.
   - Derives from `ChatUSDSupervisorNode`
   - Interacts with LLMs to determine which agent to call based on the user query
   - Can transform user queries to match agent capabilities
   - Uses system messages to understand agent capabilities and make routing decisions
   - Combines the base ChatUSDSupervisorNode system message with additional navigation-specific instructions

3. **NavigationNetworkNode**: The navigation agent implementation.
   - Derives from `NetworkNode`
   - Does not interact with LLMs directly but contains nodes that do
   - Registered as "ChatUSD_Navigation" and referenced in the route_nodes list of ChatUSDNavigationNetworkNode
   - Provides its docstring to the supervisor to explain its capabilities
   - Contains modifiers that process command outputs

4. **NavigationGenNode**: The component that generates navigation commands.
   - Derives from `RunnableNode`
   - Directly interacts with LLMs to generate navigation commands
   - Uses a specific system message that instructs the LLM to output specific command formats
   - Its outputs are intercepted by NavigationModifier
   - Its outputs are not directly visible to the supervisor, only the final result is

### Modifier's Role in Command Execution

The NavigationModifier plays a critical role in the execution of navigation commands:

1. **Command Interception**: The modifier's `on_post_invoke_async` method is called after each node in the network is invoked. It carefully checks conditions to prevent infinite loops:
   ```python
   if (
       node.invoked
       and isinstance(node.outputs, AIMessage)
       and node.outputs.content
       and not network.get_children(node)
   ):
   ```

2. **Command Processing**: When NavigationGenNode generates a command like "LIST", "NAVIGATE", or "SAVE", the modifier intercepts it and executes the corresponding action.

3. **Result Injection**: After processing a command, the modifier creates a new `RunnableHumanNode` with the result:
   ```python
   with network:
       RunnableHumanNode(f"Assistant: {result}")
   ```

4. **Continued Execution**: Creating this new node continues invoking the network. The default modifier for NavigationNetworkNode automatically creates the default node (NavigationGenNode) after the RunnableHumanNode.

5. **Command Execution in USD Stage**: The modifier executes navigation commands directly in the current USD stage. For example:
   - LIST command retrieves points of interest from the stage metadata
   - NAVIGATE command sets the camera transform to a specific position
   - SAVE command stores the current camera position in the stage metadata


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/kit-usd-agents](https://github.com/NVIDIA-Omniverse/kit-usd-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
