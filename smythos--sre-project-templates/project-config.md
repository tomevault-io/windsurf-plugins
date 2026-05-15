---
trigger: always_on
description: > **Purpose**: This document provides precise, verified guidelines for LLMs building AI agents with the SmythOS SDK. All patterns are derived from official SmythOS documentation and examples.
---

# SmythOS SDK Guidelines for LLM Contributors

> **Purpose**: This document provides precise, verified guidelines for LLMs building AI agents with the SmythOS SDK. All patterns are derived from official SmythOS documentation and examples.

> The latest version of this AGENT.md can be downloaded from https://raw.githubusercontent.com/SmythOS/sre-project-templates/refs/heads/main/AGENTS.md

## Quick Reference

```typescript
// Minimal agent setup
import { Agent, TLLMEvent } from "@smythos/sdk";

const agent = new Agent({
  name: "My Agent",
  model: "gpt-4o",
  behavior: "You are a helpful assistant.",
});

const response = await agent.prompt("Hello");
```

## Official Resources (Verify When In Doubt)

| Resource          | URL                                                             |
| ----------------- | --------------------------------------------------------------- |
| GitHub Repository | https://github.com/SmythOS/sre                                  |
| SDK Documentation | https://smythos.github.io/sre/sdk/                              |
| Code Examples     | https://github.com/SmythOS/sre/tree/main/examples               |
| Cheat Sheet       | https://smythos.github.io/sre/sdk/documents/99-cheat-sheet.html |

**If a user reports your implementation doesn't work**, check these URLs before assuming the user is wrong—the SDK evolves.

---

## 1. Import Paths

SmythOS provides two import paths for different use cases:

### Main SDK (Default)

```typescript
import { Agent, TLLMEvent } from "@smythos/sdk";
```

Use this for:

- Creating and configuring agents
- Adding skills
- Prompting and streaming
- Chat sessions

### Core SRE (Advanced)

```typescript
import { SRE, SecureConnector, ACL, TAccessLevel } from "@smythos/sdk/core";
```

Use this **only** for:

- Custom connector implementations
- Enterprise security configurations (ACL/Candidate management)
- Direct SRE runtime initialization

**Rule**: Use `@smythos/sdk` for 95% of use cases.

---

## 2. Agent Configuration

### Required Properties

```typescript
const agent = new Agent({
  name: "Customer Support Agent", // How the agent identifies itself
  model: "gpt-4o", // LLM model identifier
  behavior:
    "You are a customer support specialist for TechCorp. " +
    "Help users troubleshoot technical issues with empathy.",
});
```

| Property   | Required | Description                                                                                 |
| ---------- | -------- | ------------------------------------------------------------------------------------------- |
| `name`     | Yes      | Agent's identity name                                                                       |
| `model`    | Yes      | Model identifier string (e.g., `'gpt-4o'`, `'gpt-4o-mini'`, `'claude-3-5-sonnet-20241022'`) |
| `behavior` | Yes      | System prompt describing the agent's personality and role                                   |

### Behavior Guidelines

**DO**: Write specific, contextual behaviors

```typescript
behavior: "You are a financial analyst for hedge fund clients. " +
  "Provide data-driven insights with citations to sources. " +
  "Always express uncertainty when data is incomplete.";
```

**DON'T**: Use vague behaviors

```typescript
behavior: "helpful assistant"; // Too vague - agent lacks direction
```

### Supported Models

Models are automatically resolved through the vault system:

| Model ID            | Provider  |
| ------------------- | --------- |
| `gpt-4o`            | OpenAI    |
| `gpt-4o-mini`       | OpenAI    |
| `claude-sonnet-4-5` | Anthropic |
| `gemini-3-pro`      | Google    |

### Extended Models Directory

SmythOS scans the `.smyth/models/` directory in real-time and automatically loads any model configurations found. This directory can be located under the current project or user home (`~/.smyth/models/`).

To access a comprehensive list of supported models maintained by the SmythOS team, clone the official models repository:

```bash
# Clone into project-local models directory
git clone https://github.com/SmythOS/sre-models-pub .smyth/models/sre-models-pub

# Or clone into user home for global access
git clone https://github.com/SmythOS/sre-models-pub ~/.smyth/models/sre-models-pub
```

This repository includes configurations for models from **OpenAI**, **Anthropic**, **GoogleAI**, **Groq**, **TogetherAI**, **xAI**, and more. You can also add your own custom model configurations to this directory—SmythOS will detect and load them automatically.

---

## 3. Skills

Skills extend agent capabilities. The LLM uses the `description` field to decide when to invoke a skill.

### Skill Structure

```typescript
agent.addSkill({
  name: "get_book_info", // snake_case identifier
  description: "Get information about a book by its name", // LLM reads this
  process: async ({ book_name }) => {
    // Destructure parameters
    const url = `https://openlibrary.org/search.json?q=${book_name}`;
    const response = await fetch(url);
    const data = await response.json();
    return data.docs[0]; // Return JSON-serializable value
  },
});
```

### Skill Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmythOS/sre-project-templates](https://github.com/SmythOS/sre-project-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
