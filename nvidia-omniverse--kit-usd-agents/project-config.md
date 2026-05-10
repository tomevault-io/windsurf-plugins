---
trigger: always_on
description: Comprehensive documentation for Chat USD, a specialized AI assistant for Universal Scene Description (USD) development.
---


# Chat USD Documentation

This document contains the complete documentation for Chat USD, a specialized AI assistant for Universal Scene Description (USD) development.

## Table of Contents

- [Introduction](#README.md)
- [Overview](#Overview.md)
- [Architecture](#architecture\README.md)
- [Multi Agent Architecture](#architecture\multi-agent-architecture.md)
- [Component Interactions](#architecture\component-interactions.md)
- [Message Flow](#architecture\message-flow.md)
- [Extension Integration](#architecture\extension-integration.md)
- [Components](#components\README.md)
- [Chat Usd Network Node](#components\chat-usd-network-node.md)
- [Chat Usd Supervisor Node](#components\chat-usd-supervisor-node.md)
- [Usd Code Interactive Network Node](#components\usd-code-interactive-network-node.md)
- [Usd Search Network Node](#components\usd-search-network-node.md)
- [Scene Info Network Node](#components\scene-info-network-node.md)
- [Modifiers](#components\modifiers.md)
- [Extending](#advanced\extending.md)

---

<a id='README.md'></a>

# Introduction

# Chat USD Documentation

This documentation provides a comprehensive guide to understanding and creating agents like Chat USD, a specialized AI assistant for Universal Scene Description (USD) development.

## About Chat USD

Chat USD is a specialized AI assistant for Universal Scene Description (USD) development. It leverages the LC Agent framework to provide a multi-agent system capable of:

1. Answering knowledge-based questions about USD
2. Searching for USD assets
3. Generating and executing USD code
4. Providing scene information
5. Creating interactive UI elements with omni.ui

---

<a id='Overview.md'></a>

# Overview

# Chat USD Overview

## Introduction

Chat USD is a specialized AI assistant designed to facilitate Universal Scene Description (USD) development through natural language interaction. Built on the LC Agent framework, Chat USD provides a multi-agent system that enables users to interact with USD scenes, generate code, search for assets, and obtain information about scene elements using conversational language.

## Core Capabilities

Chat USD offers several key capabilities:

1. **USD Code Generation**: Creates and executes USD code based on natural language descriptions
2. **USD Asset Search**: Searches for USD assets based on natural language queries
3. **Scene Information Retrieval**: Analyzes and provides information about the current USD scene
4. **Interactive Development**: Enables real-time modification of USD scenes through conversation
5. **UI Integration**: Creates interactive UI elements with omni.ui (in the omni.ui variant)

## Architecture Overview

Chat USD is built on a multi-agent architecture that routes user queries to specialized agents based on the query's intent:

```
                  ┌─────────────────────┐
                  │                     │
                  │  ChatUSDNetworkNode │
                  │                     │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │                     │
                  │ChatUSDSupervisorNode│
                  │                     │
                  └──────────┬──────────┘
                             │
                             ▼
         ┌───────────────────┼───────────────────┐
         │                   │                   │
┌────────▼─────────┐ ┌───────▼────────┐ ┌────────▼─────────┐
│                  │ │                │ │                  │
│USDCodeInteractive│ │   USDSearch    │ │    SceneInfo     │
│     NetworkNode  │ │  NetworkNode   │ │   NetworkNode    │
│                  │ │                │ │                  │
└──────────────────┘ └────────────────┘ └──────────────────┘
```

## Key Components

### ChatUSDNetworkNode
- Main entry point for user interactions
- Routes queries to appropriate specialized agents
- Coordinates responses from multiple agents
- Presents final results to the user

### ChatUSDSupervisorNode
- Orchestrates the multi-agent system
- Determines which specialized agent should handle a query
- Formulates appropriate sub-queries for each agent
- Integrates responses from multiple agents

### USDCodeInteractiveNetworkNode
- Generates USD code based on natural language descriptions
- Executes code to modify the USD scene
- Validates and fixes code issues
- Provides feedback on code execution

### USDSearchNetworkNode
- Interprets natural language search queries
- Searches for relevant USD assets
- Presents search results with previews
- Facilitates asset import into the scene

### SceneInfoNetworkNode
- Analyzes the current USD scene
- Extracts relevant information about scene elements
- Provides context for other agents
- Answers queries about scene structure and properties

---

<a id='architecture\README.md'></a>

# Architecture

# Chat USD Architecture

This section provides a detailed overview of the Chat USD architecture, explaining how the different components work together to create a powerful USD development assistant.

## Architecture Overview


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/kit-usd-agents](https://github.com/NVIDIA-Omniverse/kit-usd-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
