---
trigger: always_on
description: Defines the three agent archetypes (participant, mentor, judge), their seniority tiers, tools, system prompts, and how to generate the cast.
---

# Hackathon Simulation — Agent Types

Defines the three agent archetypes (participant, mentor, judge), their seniority tiers, tools, system prompts, and how to generate the cast.

See `SKILLS.md` for how to spawn and run.

---

## Winning Patterns from Knowledge Base (260+ Hackathons)

Based on analysis of actual hackathon winners:

| Pattern | Domains | Why It Wins |
|---------|---------|-------------|
| **AI Agents** | All (esp. Web3, AI/ML) | 2024-2025 dominant - automation + LLM integration |
| **Sponsor Integration** | Web3, FinTech, AI/ML | **#1 predictor** - winners use 2+ sponsor APIs |
| **Real-world Impact** | Healthcare, Civic, Climate | Solves actual problems, not just tech demos |
| **Accessibility** | Healthcare, Civic, EdTech | High impact, clear user need |
| **Non-developer + AI** | All (Claude Code example) | Domain experts with AI tools win |
| **Edge AI / Local Inference** | Healthcare, IoT | Privacy + offline capability |
| **IoT/Sensors** | Healthcare, Climate, Hardware | Real data, physical impact |

### Domain-Specific Winners

- **Healthcare**: AI diagnostics, wearables, EHR integration, mental health apps
- **Web3/Blockchain**: Account abstraction (smart wallets), ZK/privacy, consumer DeFi
- **FinTech**: Cross-border payments, accessible banking, embedded finance
- **EdTech**: Accessibility tools, AI tutoring, skills assessment
- **Climate**: Carbon tracking, energy optimization, agriculture tech

---

## Quick Reference

| Role | Purpose | Tick Active | MCP |
|------|--------|-----------|-----|
| `participant` | Builds and pitches projects | 1-48 | Brave Search (if junior) |
| `mentor` | Guides, probes, refines, VERIFIES between ticks | 1-48 | Brave Search (always) |
| `judge` | Scores, debates, selects | 47-48 | Brave Search (always) |

---

## Shared Tools

### `broadcast_message`
```typescript
{
  name: "broadcast_message",
  description: "Broadcast a message to all hackathon participants.",
  inputSchema: {
    type: "object",
    properties: {
      content: { type: "string", description: "Message to broadcast" },
    },
    required: ["content"],
  },
  async execute(input, ctx) { broadcastToAll(ctx, input.content); },
}
```

### `send_message`
```typescript
{
  name: "send_message",
  description: "Send a private message to a specific agent.",
  inputSchema: {
    type: "object",
    properties: {
      recipientId: { type: "string", description: "Target agent ID" },
      content: { type: "string", description: "The message" },
    },
    required: ["recipientId", "content"],
  },
  async execute(input, ctx) { sendTo(ctx, input.recipientId, input.content); },
}
```

### `check_agent_status`
```typescript
{
  name: "check_agent_status",
  description: "Check which agents are active or what teams have formed.",
  inputSchema: { type: "object", properties: {}, required: [] },
  async execute(_, ctx) { return listActiveAgents(ctx); },
}
```

---

## Web Search (MCP)

Via Brave Search at `http://localhost:3001`:
```
mcp_brave_search_search
  input:  { query: string, count?: number }
  output: Search results with title, url, snippet
```

### When to Use Web Search

| Trigger | Search Query Example | Purpose |
|---------|---------------------|---------|
| **Tech verification** | "LLM code review tools 2024" | Verify technology claims |
| **Domain research** | "healthcare hackathon winners 2024" | Find winning patterns |
| **Competitor analysis** | "AI accessibility tools existing" | Differentiate your idea |
| **Sponsor integration** | "GitHub API integration tutorial" | Learn sponsor APIs |
| **Trend checking** | "account abstraction wallet trends" | Validate idea timing |
| **Real-world validation** | "diabetes monitoring apps FDA" | Confirm real problem |

### Web Search Prompts by Agent

**Participant prompts:**
```
Before finalizing your idea, search for:
1. "What existing solutions solve [your problem]?"
2. "What [theme] hackathon winners 2024?"
3. "[Your technology] vs alternatives comparison"
```

**Mentor prompts:**
```
When giving feedback, search to verify:
1. "Is [team's technology claim] accurate?"
2. "[Domain] winning projects at recent hackathons"
3. "Sponsor APIs available for [theme]"
```

**Judge prompts:**
```
Before scoring, search to validate:
1. "Does [project claim] have evidence?"
2. "[Theme] hackathon trends 2024"
3. "Who are the competitors for [project type]?"
```

Config:
```typescript
mcp: [{
  name: "brave-search",
  transport: "http",
  url: "http://localhost:3001",
}]
```

---

## 1. Participant

The core builder. Generates ideas, forms teams, builds pitches.

### Seniority Tiers

| Tier | Experience | Tech Confidence | MCP |
|------|-----------|----------------|-----|
| `junior` | 0-2 years | Low — needs search to vet tech | Brave Search |
| `mid` | 2-5 years | Moderate | Brave Search |
| `senior` | 5+ years | High — may skip search | None |

### Participant Prompt Template

```
You are {name}, a {seniority} {type} participant at a hackathon.
Theme: "{theme}"

Your expertise: {expertise_list}
Your personality: {personality_traits}

Based on analysis of 260+ winning hackathon projects, these patterns WIN:
- AI Agents: LLM-powered automation (dominant 2024-2025)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Spirizeon/hackfish](https://github.com/Spirizeon/hackfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
