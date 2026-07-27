---
trigger: always_on
description: title: "Building Agents"
---

---
title: "Building Agents"
description: "ReAct patterns, orchestration, and memory management"
summary: "Create intelligent agents with reasoning, tool use, and conversation memory"
date: 2025-10-13T00:00:00+00:00
lastmod: 2025-10-13T00:00:00+00:00
draft: false
weight: 500
toc: true
seo:
  title: "Building Agents - dspy-go"
  description: "Complete guide to building agents with ReAct, orchestration, and memory in dspy-go"
  canonical: ""
  noindex: false
---

# Building Agents

dspy-go's **agent package** provides powerful abstractions for building intelligent agents that can reason, use tools, maintain conversation history, and orchestrate complex workflows.

## Agent Architecture

An agent in dspy-go combines:
- **ReAct Module**: Reasoning + Acting pattern
- **Tool Registry**: Available tools the agent can use
- **Memory**: Conversation history and context
- **Orchestrator**: Task decomposition and coordination

---

## ReAct Pattern

**Reasoning and Acting** - The foundation of intelligent agents.

### What is ReAct?

ReAct combines:
1. **Thought**: The agent reasons about what to do
2. **Action**: The agent uses a tool
3. **Observation**: The agent sees the tool's result
4. **Repeat**: Until the task is complete

### Basic ReAct Agent

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/XiaoConstantine/dspy-go/pkg/core"
    "github.com/XiaoConstantine/dspy-go/pkg/modules"
    "github.com/XiaoConstantine/dspy-go/pkg/tools"
)

func main() {
    // Configure LLM
    llm, _ := llms.NewGeminiLLM("", core.ModelGoogleGeminiPro)
    core.SetDefaultLLM(llm)

    // Create tools
    calculator := tools.NewCalculatorTool()
    searchTool := tools.NewSearchTool()
    weatherTool := tools.NewWeatherTool()

    // Create tool registry
    registry := tools.NewInMemoryToolRegistry()
    registry.Register(calculator)
    registry.Register(searchTool)
    registry.Register(weatherTool)

    // Define signature for the agent's task
    signature := core.NewSignature(
        []core.InputField{
            {Field: core.NewField("question",
                core.WithDescription("The question to answer"))},
        },
        []core.OutputField{
            {Field: core.NewField("answer",
                core.WithDescription("The final answer"))},
        },
    )

    // Create ReAct module
    react := modules.NewReAct(
        signature,
        registry,
        5, // max iterations
    )

    // Execute
    ctx := context.Background()
    result, err := react.Process(ctx, map[string]interface{}{
        "question": "What is the population of Tokyo divided by 1000?",
    })
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Answer: %s\n", result["answer"])
}
```

### How ReAct Works

Given the question "What is the population of Tokyo divided by 1000?":

**Iteration 1:**
- **Thought**: "I need to find the population of Tokyo"
- **Action**: `search("Tokyo population")`
- **Observation**: "Tokyo has a population of approximately 14 million"

**Iteration 2:**
- **Thought**: "Now I need to divide 14,000,000 by 1000"
- **Action**: `calculator("14000000 / 1000")`
- **Observation**: "14000"

**Iteration 3:**
- **Thought**: "I have the answer"
- **Action**: `finish("14,000")`

---

## Custom Tools

**Extend agents** with domain-specific tools.

### Creating a Custom Tool

```go
package main

import (
    "context"
    "fmt"
    "strings"
)

// Custom Weather Tool
type WeatherTool struct{}

func (t *WeatherTool) GetName() string {
    return "weather"
}

func (t *WeatherTool) GetDescription() string {
    return "Get the current weather for a location. Usage: weather(location)"
}

func (t *WeatherTool) CanHandle(action string) bool {
    return strings.HasPrefix(action, "weather(")
}

func (t *WeatherTool) Execute(ctx context.Context, action string) (string, error) {
    // Parse location from action string
    location := parseLocation(action)

    // Fetch weather data (your implementation)
    weather, err := fetchWeather(location)
    if err != nil {
        return "", err
    }

    return fmt.Sprintf("Weather in %s: %s, %d°C",
        location, weather.Condition, weather.Temperature), nil
}

func parseLocation(action string) string {
    // Extract "Paris" from "weather(Paris)"
    start := strings.Index(action, "(") + 1
    end := strings.Index(action, ")")
    return action[start:end]
}
```

### Using Custom Tools

```go
// Register custom tool
registry := tools.NewInMemoryToolRegistry()
registry.Register(&WeatherTool{})
registry.Register(&DatabaseTool{})
registry.Register(&EmailTool{})

// Create ReAct agent with custom tools
react := modules.NewReAct(signature, registry, 10)
```

**[Full Agents Example →](https://github.com/XiaoConstantine/dspy-go/tree/main/examples/agents)**

---

## Memory Management

**Conversation history** and context tracking.

### Buffer Memory

Store recent conversation turns:

```go
package main

import (
    "context"
    "github.com/XiaoConstantine/dspy-go/pkg/agents/memory"
)

func main() {
    // Create buffer memory (keeps last 10 exchanges)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XiaoConstantine/dspy-go](https://github.com/XiaoConstantine/dspy-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
