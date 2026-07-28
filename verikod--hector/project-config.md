---
trigger: always_on
description: Agents are the core building blocks of Hector. An **Agent** is an autonomous entity that combines an LLM, Tools, and Instructions to solve tasks.
---

# Agents

Agents are the core building blocks of Hector. An **Agent** is an autonomous entity that combines an LLM, Tools, and Instructions to solve tasks.

## Agent Types

| Type | Description |
|------|-------------|
| `llm` | LLM-backed intelligent agent (default) |
| `sequential` | Runs sub-agents in order |
| `parallel` | Runs sub-agents concurrently |
| `loop` | Iterates until condition met |
| `conditional` | Routes based on evaluation |
| `runner` | Deterministic tool pipeline (no LLM) |
| `remote` | Proxies to external A2A agent |

## LLM Agents

The default agent type uses an LLM for reasoning.

```yaml
agents:
  assistant:
    name: "Assistant"
    description: "A helpful AI assistant"
    llm: claude
    instruction: "You are a helpful assistant."
    tools: [search, calculator]
```

### Key Components

| Component | Description |
|-----------|-------------|
| **llm** | Model powering the agent (e.g., `claude`, `gpt4`) |
| **instruction** | System prompt defining behavior |
| **tools** | Capabilities the agent can call |

### Reasoning Loop

When an agent receives a task:

1. **Observe**: Read conversation history and input
2. **Think**: LLM generates decision
3. **Act**: Emit tool call if needed
4. **Result**: Execute tool, return result
5. **Repeat**: Until final answer

### Instructions

Inline or from file:

```yaml
# Inline
instruction: "You are a concise chatbot."

# From file
instruction_file: "./prompts/researcher.md"
```

Template variables for dynamic context:
- `{user:name}` - User-scoped
- `{app:config}` - App-scoped
- `{artifact.data}` - File content

### Reasoning Configuration

```yaml
agents:
  coder:
    reasoning:
      max_iterations: 50
      enable_exit_tool: true
      enable_escalate_tool: true
```

### Context & Memory

Control conversation history to fit LLM context limits. See the [Context & Memory Strategies](#context-memory-strategies) section below for full details.

```yaml
agents:
  chatbot:
    context:
      strategy: token_window
      budget: 8000
```

---

## Multi-Agent Orchestration

Compose agents into complex systems using workflow types.

### Sequential

Run sub-agents in strict order:

```yaml
agents:
  blog_pipeline:
    type: sequential
    sub_agents: [researcher, writer, editor]

  researcher:
    llm: claude
    instruction: "Find facts about the topic."
  writer:
    llm: claude
    instruction: "Write a draft."
  editor:
    llm: gpt4
    instruction: "Fix grammar and tone."
```

### Parallel

Run sub-agents concurrently:

```yaml
agents:
  consensus:
    type: parallel
    sub_agents: [analyst_a, analyst_b, analyst_c]
```

### Loop

Iterate until condition or max:

```yaml
agents:
  refinement:
    type: loop
    sub_agents: [coder, reviewer]
    max_iterations: 3
```

### Conditional

Route based on evaluation:

```yaml
agents:
  safe_assistant:
    type: conditional
    condition_agent: moderator
    condition_field: "safe"
    on_true_agent: helper
    on_false_response: "I cannot help with that."
```

---

## Runner Agents

Deterministic tool pipelines with **no LLM involvement**.

```yaml
agents:
  etl_job:
    type: runner
    tools: [fetch_api, transform, save_data]
```

### How It Works

1. Input parsed as JSON
2. Tool 1 receives input, returns output
3. Output of Tool N becomes input of Tool N+1
4. Final tool output is the agent response

### Use Cases

- Data fetching pipelines
- ETL workflows
- CI/CD automation
- Format conversion

### Combining with LLM Agents

Use runners as sub-agents for reliable data fetching:

```yaml
agents:
  analyst:
    llm: claude
    sub_agents: [data_fetcher]

  data_fetcher:
    type: runner
    tools: [stock_api, news_api]
```

---

## Agent Composition

### Sub-Agents (Transfer)

Control transfers to sub-agent. The parent hands off the conversation. The sub-agent takes over, including the user interaction.

```yaml
agents:
  manager:
    sub_agents: [researcher, writer]
```

Runtime provides `transfer_to_researcher` and `transfer_to_writer` tools.

### Agent Tools (Delegation)

Parent stays in control. The child agent executes in an isolated session and returns a result. The parent decides what to do with it.

```yaml
agents:
  assistant:
    agent_tools: [fact_checker]
```

### When to Use Which

| | Sub-Agents (Transfer) | Agent Tools (Delegation) |
|-|-----------------------|-------------------------|
| **Control** | Child takes over conversation | Parent stays in control |
| **Session** | Shared session with parent | Isolated session (no state bleed) |
| **Best for** | Routing/triage, specialized handlers | Helper tasks, data enrichment |
| **Example** | "Transfer to billing department" | "Ask the fact-checker about this claim" |

**Rule of thumb:** Use `sub_agents` when the child should talk directly to the user. Use `agent_tools` when the parent needs to process the child's output.

---

## Triggers

Run agents automatically.

### Scheduled

```yaml
trigger:
  type: schedule
  cron: "0 9 * * *"
  timezone: America/New_York
```

### Webhook

```yaml
trigger:
  type: webhook
  path: /webhooks/github
  secret: ${GITHUB_SECRET}
```

See [Triggers Guide](./triggers.md) for details.

---

## Notifications


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verikod/hector](https://github.com/verikod/hector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
