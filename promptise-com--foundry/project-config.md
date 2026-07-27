---
trigger: always_on
description: Core agent creation, invocation, identity, conversations, and the full reasoning engine — every public class across `promptise.agent`, `promptise.engine`, and `promptise.conversations`.
---

# Agent API Reference

Core agent creation, invocation, identity, conversations, and the full reasoning engine — every public class across `promptise.agent`, `promptise.engine`, and `promptise.conversations`.

## Building Agents

### build_agent

::: promptise.agent.build_agent
    options:
      show_source: false
      heading_level: 3

### PromptiseAgent

::: promptise.agent.PromptiseAgent
    options:
      show_source: false
      heading_level: 3
      members:
        - ainvoke
        - astream
        - astream_with_tools
        - invoke
        - chat
        - get_session
        - list_sessions
        - delete_session
        - update_session
        - shutdown
        - get_stats
        - generate_report

### CallerContext

::: promptise.agent.CallerContext
    options:
      show_source: false
      heading_level: 3

### get_current_caller

::: promptise.agent.get_current_caller
    options:
      show_source: false
      heading_level: 3

---

## Reasoning Graph Engine

### PromptGraphEngine

::: promptise.engine.execution.PromptGraphEngine
    options:
      show_source: false
      heading_level: 4
      members:
        - ainvoke
        - astream_events
        - last_report

### PromptGraph

::: promptise.engine.graph.PromptGraph
    options:
      show_source: false
      heading_level: 4

### Edge

::: promptise.engine.graph.Edge
    options:
      show_source: false
      heading_level: 4

---

## Graph State

### GraphState

::: promptise.engine.state.GraphState
    options:
      show_source: false
      heading_level: 4

### NodeResult

::: promptise.engine.state.NodeResult
    options:
      show_source: false
      heading_level: 4

### NodeEvent

::: promptise.engine.state.NodeEvent
    options:
      show_source: false
      heading_level: 4

### NodeFlag

::: promptise.engine.state.NodeFlag
    options:
      show_source: false
      heading_level: 4

### GraphMutation

::: promptise.engine.state.GraphMutation
    options:
      show_source: false
      heading_level: 4

### ExecutionReport

::: promptise.engine.state.ExecutionReport
    options:
      show_source: false
      heading_level: 4

---

## Base Node Types

### BaseNode

::: promptise.engine.base.BaseNode
    options:
      show_source: false
      heading_level: 4

### NodeProtocol

::: promptise.engine.base.NodeProtocol
    options:
      show_source: false
      heading_level: 4

### @node decorator

::: promptise.engine.base.node
    options:
      show_source: false
      heading_level: 4

---

## Standard Nodes

### PromptNode

::: promptise.engine.nodes.PromptNode
    options:
      show_source: false
      heading_level: 4

### ToolNode

::: promptise.engine.nodes.ToolNode
    options:
      show_source: false
      heading_level: 4

### RouterNode

::: promptise.engine.nodes.RouterNode
    options:
      show_source: false
      heading_level: 4

### GuardNode

::: promptise.engine.nodes.GuardNode
    options:
      show_source: false
      heading_level: 4

### ParallelNode

::: promptise.engine.nodes.ParallelNode
    options:
      show_source: false
      heading_level: 4

### LoopNode

::: promptise.engine.nodes.LoopNode
    options:
      show_source: false
      heading_level: 4

### HumanNode

::: promptise.engine.nodes.HumanNode
    options:
      show_source: false
      heading_level: 4

### TransformNode

::: promptise.engine.nodes.TransformNode
    options:
      show_source: false
      heading_level: 4

### SubgraphNode

::: promptise.engine.nodes.SubgraphNode
    options:
      show_source: false
      heading_level: 4

### AutonomousNode

::: promptise.engine.nodes.AutonomousNode
    options:
      show_source: false
      heading_level: 4

### CodeActionNode

::: promptise.engine.code_action.CodeActionNode
    options:
      show_source: false
      heading_level: 4

---

## Reasoning Nodes

### ThinkNode

::: promptise.engine.reasoning_nodes.ThinkNode
    options:
      show_source: false
      heading_level: 4

### PlanNode

::: promptise.engine.reasoning_nodes.PlanNode
    options:
      show_source: false
      heading_level: 4

### ReflectNode

::: promptise.engine.reasoning_nodes.ReflectNode
    options:
      show_source: false
      heading_level: 4

### CritiqueNode

::: promptise.engine.reasoning_nodes.CritiqueNode
    options:
      show_source: false
      heading_level: 4

### ObserveNode

::: promptise.engine.reasoning_nodes.ObserveNode
    options:
      show_source: false
      heading_level: 4

### SynthesizeNode

::: promptise.engine.reasoning_nodes.SynthesizeNode
    options:
      show_source: false
      heading_level: 4

### ValidateNode

::: promptise.engine.reasoning_nodes.ValidateNode
    options:
      show_source: false
      heading_level: 4

### JustifyNode

::: promptise.engine.reasoning_nodes.JustifyNode
    options:
      show_source: false
      heading_level: 4

### RetryNode

::: promptise.engine.reasoning_nodes.RetryNode
    options:
      show_source: false
      heading_level: 4

### FanOutNode

::: promptise.engine.reasoning_nodes.FanOutNode
    options:
      show_source: false
      heading_level: 4

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [promptise-com/Foundry](https://github.com/promptise-com/Foundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
