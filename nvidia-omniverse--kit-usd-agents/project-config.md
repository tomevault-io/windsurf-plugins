---
trigger: always_on
description: A high-level overview of the core components in LC Agent USD module, explaining how they work together to create a specialized system for USD development assistance.
---


# LC Agent USD Module Overview

## Introduction

The `lc_agent_usd` module is a specialized extension of the LC Agent framework designed to provide intelligent assistance for Universal Scene Description (USD) development. It leverages the core LC Agent architecture to create a system that can:

1. Answer knowledge-based questions about USD
2. Generate and validate USD code snippets
3. Provide interactive code assistance
4. Execute and debug USD code

This module demonstrates how LC Agent's flexible architecture can be extended to create domain-specific AI assistants with specialized capabilities.

## Core Components

The module is organized into several key components:

### Network Nodes

Network nodes are specialized classes that extend the `NetworkNode` base class from LC Agent. They serve as containers for specific functionality:

- **USDKnowledgeNetworkNode**: Provides factual information about USD concepts and usage
- **USDCodeNetworkNode**: Handles general USD code generation and validation
- **USDCodeGenNetworkNode**: Specializes in generating executable USD code with validation

### Specialized Nodes

These are the building blocks that implement specific behaviors:

- **USDKnowledgeNode**: Processes knowledge-based queries about USD
- **USDCodeGenNode**: Generates USD code snippets with proper structure
- **USDCodeInteractiveNode**: Provides interactive code assistance

### Modifiers

Modifiers extend the functionality of nodes by intercepting and modifying their behavior:

- **USDKnowledgeRagModifier**: Enhances knowledge responses with retrieval-augmented generation
- **USDCodeGenRagModifier**: Enhances code generation with retrieval-augmented generation
- **CodeInterpreterModifier**: Executes and validates code
- **CodeExtractorModifier**: Extracts and formats code snippets
- **USDCodeGenPatcherModifier**: Fixes and improves generated code

## System Architecture

The module follows a layered architecture:

1. **User Interface Layer**: Receives queries and displays responses
2. **Network Layer**: Routes queries to appropriate specialized nodes
3. **Processing Layer**: Generates responses using specialized nodes
4. **Modifier Layer**: Enhances responses with additional capabilities
5. **Knowledge Layer**: Retrieves relevant information from knowledge bases

## Integration with LC Agent Core

The `lc_agent_usd` module integrates with the core LC Agent framework by:

1. Extending base classes like `NetworkNode` and `RunnableNode`
2. Implementing custom modifiers that work with the LC Agent modifier system
3. Registering custom node types with the node factory
4. Using the LC Agent message passing system for communication

## Use Cases

The module is designed to support several key use cases:

1. **USD Knowledge Assistance**: Answering questions about USD concepts, API, and best practices
2. **Code Generation**: Creating USD code snippets based on user requirements
3. **Code Validation**: Checking and fixing USD code for correctness
4. **Interactive Development**: Providing real-time assistance during USD development


# USDKnowledgeNetworkNode

## Overview

The `USDKnowledgeNetworkNode` is a specialized network node in the LC Agent USD module that provides knowledge-based assistance for Universal Scene Description (USD). It serves as a comprehensive information source for USD concepts, API usage, best practices, and general questions about USD functionality.

## Purpose

The primary purpose of the `USDKnowledgeNetworkNode` is to:

1. Answer factual questions about USD concepts and terminology
2. Provide explanations of USD API functions and classes
3. Offer guidance on USD best practices and workflows
4. Explain USD file formats and structure
5. Assist with understanding USD's role in 3D pipelines

This node is designed to be the knowledge foundation of the USD agent system, focusing on providing accurate information rather than generating or executing code.

## Implementation Details

### Class Definition

```python
class USDKnowledgeNetworkNode(NetworkNode):
    default_node: str = "USDKnowledgeNode"

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.add_modifier(USDKnowledgeRagModifier())
```

The implementation is intentionally simple, leveraging the power of the LC Agent framework to handle most of the complexity. The class:

1. Extends `NetworkNode` from the core LC Agent framework
2. Sets `USDKnowledgeNode` as its default node type
3. Adds a `USDKnowledgeRagModifier` to enhance responses with retrieval-augmented generation

### Default Node

The `USDKnowledgeNode` serves as the default processing node for this network. It:

1. Processes queries using a system message to generate appropriate responses
2. Handles knowledge-based questions about USD
3. Provides factual information about USD concepts and API

### RAG Modifier

The `USDKnowledgeRagModifier` enhances the node's responses by:

1. Retrieving relevant information from a knowledge base of USD documentation
2. Injecting this information into the context before generating responses
3. Ensuring responses are grounded in accurate USD documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/kit-usd-agents](https://github.com/NVIDIA-Omniverse/kit-usd-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
