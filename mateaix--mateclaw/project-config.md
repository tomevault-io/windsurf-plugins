---
trigger: always_on
description: MateClaw's multi-agent system runs in two modes — ReAct for real-time reasoning and Plan-and-Execute for complex task decomposition. Agents can delegate to one another for true multi-agent collaboration.
---


# Multi-Agent Engine

> **They're called "digital employees" now.** The back office uses that term throughout. The runtime is still an Agent under the hood, but the UI, the mental model, and the templates treat each one as a coworker on your team.
> The renaming brings a worldview shift with it: you give an employee a **Role**, a **Goal**, and a **Backstory** — they know who they are and why they exist. You don't have to write a cold system prompt asking an "agent" to please understand the task.

An employee is a personality with tools. Multiple employees form a team.

That's the short version. The longer one: an employee is a name, a system prompt that defines how it thinks (built from role / goal / backstory), a model that actually thinks, a set of tools it's allowed to reach for, optional knowledge bases it can read, optional skills that extend what it can do, its own slice of memory, and a choice of how to approach hard problems — incrementally (ReAct) or with a plan (Plan-and-Execute).

You can have many employees. Each one is specialized. You give them different jobs.

---

## What a digital employee has

| Piece | What it is |
|-------|-----------|
| **Name** | How you and your team find them |
| **Icon** | Pixel-art style, color coded by role |
| **Role** | One sentence — "I'm the product researcher" / "I'm customer support" |
| **Goal** | One sentence — "I help you see how the market is moving" |
| **Backstory** | Where they came from, why they exist, what they care about; auto-spliced into the final system prompt |
| **Employee-card tagline** | The "self-introduction" shown on the card |
| **System prompt** | Their personality, rules, style, priorities (role/goal/backstory inject automatically) |
| **Type** | `react` or `plan_execute` |
| **Tools** | Which tools they're allowed to call (built-in, MCP, skills, ACP-bridged) |
| **Knowledge bases** | LLM Wikis they can read from (KB hot cache auto-injects into the system prompt) |
| **Workspace memory** | Their own `PROFILE.md`, `MEMORY.md`, `SOUL.md`, `AGENTS.md`, and daily notes |
| **Max iterations** | How many reasoning loops are allowed before forced convergence |
| **Enabled flag** | Off switch |

Models resolve in this order: **conversation pin → agent model override → global default**. A conversation may temporarily pin an enabled provider/model, and an agent may select its own primary model; a blank agent choice inherits the global default. If a pin is later disabled or removed, runtime falls back to the agent override or global default instead of failing the conversation. Each agent can also maintain an ordered provider/model failover preference chain.

---

## Templates: hire a coworker who already knows the job

You do not have to start from scratch. `Digital Employees → New` opens a picker populated from the server's `classpath:templates/*.json`; you can also skip templates and continue to the blank form.

### 6 built-in templates (recommended)

Each one ships with a role, goal, backstory, the right toolset, a pixel-art avatar, and a color that belongs to the role. **Open one, it works:**

- **General Assistant** — search, writing, analysis, and everyday work
- **Product Assistant** — clarify users, scenarios, and requirements before shaping product decisions
- **Research Analyst** — decompose complex research with web search and Wiki context
- **Customer Support** — empathize, search available knowledge, solve or escalate
- **Data Analyst** — query datasources, run SQL, build charts, write conclusions
- **Code Reviewer** — inspect code, identify issues, and recommend improvements

Selecting a template creates the corresponding agent immediately; skipping opens the fully editable custom form. Name, role, goal, model, skills, tools, and knowledge-base scope remain editable after creation.

---

## Two ways of thinking

### ReAct — think, act, observe, continue

The default. An agent in ReAct mode runs a loop: **reason** about what to do next, **act** (maybe by calling a tool), **observe** the result, decide whether to loop again or answer.

Use it for:
- simple Q&A that might need one or two tool calls
- conversational interaction where each user turn is small
- tasks where the agent needs to react to what it learns along the way

Example: *"What's the weather in Beijing today?"* → reason (need current data), act (call web search), observe (15–26°C, sunny), answer.

### Plan-and-Execute — plan first, execute second

For larger tasks. The agent starts by generating a **plan** — an ordered list of 2 to 6 steps. Then it executes each step, one at a time. When done, it summarizes everything it did.

Use it for:
- multi-step research ("investigate X, compare Y, write a brief")
- anything where the steps are knowable up front
- anything where you want to **watch progress** — the plan and each step's status show up in a persistent task list next to the conversation

Example: *"Research Spring AI frameworks, compare the top three, write me a brief."* → plan (4 steps) → execute in order → summarize.

### How to choose

| Situation | Use | Why |
|-----------|-----|-----|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mateaix/mateclaw](https://github.com/mateaix/mateclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
