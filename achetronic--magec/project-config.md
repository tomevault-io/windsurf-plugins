---
trigger: always_on
description: An agent is the core building block of Magec. Each agent is an independent AI entity with its own LLM, personality, memory, voice, and tools. You can think of an agent as a specialized worker — you define who it is, what it knows, what it can do, and how it communicates.
---


An agent is the core building block of Magec. Each agent is an independent AI entity with its own LLM, personality, memory, voice, and tools. You can think of an agent as a specialized worker — you define who it is, what it knows, what it can do, and how it communicates.

Agents are created and managed entirely from the Admin UI. No config files, no code. Click, configure, save — the agent is live immediately.

<div class="screenshots" style="margin-bottom: 2rem;">
{{< screenshot src="img/screenshots/admin-agents.png" alt="Admin UI — Agents list" >}}
</div>

Click any agent to open its configuration. The settings are organized in collapsible sections so you can focus on what matters.

<div class="screenshots" style="margin-bottom: 2rem;">
{{< screenshot src="img/screenshots/admin-agent-dialog.png" alt="Admin UI — New Agent dialog" >}}
</div>

## General

The basics: name, description, and tags.

The **name** is how this agent appears everywhere in the platform — in the Voice UI agent switcher, in flow step labels, in conversation logs, in the Telegram `/agent` command. Pick something meaningful.

The **description** is a note for yourself. It appears in the admin panel to help you remember what this agent is for when you have many of them.

**Tags** are optional labels for organizing agents. They become useful when you have dozens of agents and want to filter or group them by purpose (e.g., `customer-service`, `internal`, `research`).

| Field | Description |
|-------|-------------|
| `name` | Display name shown across the entire platform |
| `description` | Optional note for your reference |
| `tags` | Labels for filtering and grouping |

## System Prompt

This is the most important field. The system prompt defines who the agent is, how it behaves, what it should and shouldn't do, and in what style it should respond. Every single response the agent produces is shaped by this prompt.

Write the prompt as instructions to the agent. You can be as detailed as you want — multi-paragraph instructions, examples of desired output, rules, personality traits, language preferences. The more specific you are, the more predictable the agent's behavior.

**Example prompts:**

- *"You are a home automation assistant. You control lights, thermostats, and appliances via the Home Assistant MCP tools. Always confirm actions before executing them. Respond in Spanish."*
- *"You are a code reviewer. Analyze the code provided and give constructive feedback focusing on readability, performance, and potential bugs. Be direct but not rude."*
- *"You are a restaurant concierge. You know the menu, daily specials, and allergen information. When customers ask about dishes, be enthusiastic but honest."*

If you leave the system prompt empty, Magec uses a default prompt that makes the agent a general-purpose assistant.

### Output Key

The **output key** is used when the agent participates in a [flow](/docs/flows/). It saves the agent's output under a named key in the flow's shared state. Other agents in the same flow can then reference that output using `{{agent.output:key_name}}` in their own prompts.

For example, if a "researcher" agent has `outputKey: research_results`, a later "writer" agent can include `{{agent.output:research_results}}` in its prompt to access what the researcher found.

| Field | Description |
|-------|-------------|
| `systemPrompt` | The full instruction text. Supports multi-line, markdown, examples, and any formatting you want. |
| `outputKey` | Named key for flow data passing. Other agents in the same flow can reference it with `{{agent.output:key_name}}`. |

## LLM

Which AI brain powers this agent. You select a [backend](/docs/backends/) (the provider connection) and a model (which specific model to use from that provider).

The backend dropdown shows all backends you've configured. The model is a free-text field — you type the model identifier exactly as the provider expects it.

| Field | Description |
|-------|-------------|
| `llmBackend` | The AI backend to use (selected from your configured backends) |
| `llmModel` | Model identifier — e.g., `gpt-4.1`, `claude-sonnet-4-20250514`, `qwen3:8b`, `gemini-2.0-flash` |

{{< callout type="info" >}}
Each agent chooses its own backend and model independently. In a flow, you can have one agent running GPT-4 for complex reasoning and another running a fast local model for simple formatting — optimizing cost and speed at the agent level.
{{< /callout >}}

## Memory

Memory lets agents remember things between conversations. Without memory, every conversation starts from scratch. With memory, agents can recall what you talked about yesterday, remember your preferences, and build context over time.

Memory in Magec is configured **globally** — you set up memory providers once (under **Memory** in the Admin UI), and all agents automatically benefit from them. There's no per-agent memory selection.

### How it works


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achetronic/magec](https://github.com/achetronic/magec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
