---
trigger: always_on
description: Validates answers for correctness and structures them into memory objects with metadata.
---

# Agent Types in OrKa

> **Last Updated:** 03 January 2026  
> **Status:** 🟢 Current  
> **Related:** [Advanced Agents](agents-advanced.md) | [Extending Agents](extending-agents.md) | [Agent Index](AGENT_NODE_TOOL_INDEX.md) | [INDEX](index.md)

In OrKa, **agents** are modular processing units that receive input and return structured output — all orchestrated via a declarative YAML configuration.

Agents can represent different cognitive functions: classification, decision-making, web search, conditional routing, memory management, and more.

The OrKa framework uses a unified agent base implementation that supports both modern asynchronous patterns and legacy synchronous patterns for backward compatibility.

---

## 🧱 Core Agent Types

### � `brain`

Procedural skill memory — learns abstract, transferable skills from LLM reasoning traces and re-applies them across domains.

**Use case:** Cross-domain knowledge transfer, continuous learning, skill accumulation.

**Operations:**
- `learn` — Extract a transferable skill from an execution trace
- `recall` — Find applicable skills for a new context
- `feedback` — Record whether a transferred skill succeeded

**Example config:**
```yaml
- id: brain_learn
  type: brain
  operation: learn
  prompt: "{{ previous_outputs.llm_reasoner }}"

- id: brain_recall
  type: brain
  operation: recall
  prompt: "{{ previous_outputs.new_context }}"

- id: brain_feedback
  type: brain
  operation: feedback
  prompt: "{{ previous_outputs.brain_recall }}"
```

**📖 [Complete Brain Documentation](./BRAIN_SYSTEM_GUIDE.md)**

### �🧭 `graph-scout`

Intelligent workflow graph inspection and optimal multi-agent path execution. GraphScout automatically discovers, evaluates, and executes the best sequence of agents for any given input.

**Use case:** Dynamic routing, intelligent workflow orchestration, adaptive agent selection.

**Key Features:**
- **Intelligent Path Discovery**: Automatically finds optimal agent sequences
- **Memory-Aware Routing**: Positions memory agents optimally (readers first, writers last)
- **Multi-Agent Execution**: Executes ALL agents in shortlist sequentially
- **LLM-Powered Evaluation**: Advanced reasoning for path selection
- **Budget & Safety Control**: Respects token/latency budgets and safety thresholds

**Example config:**
```yaml
- id: smart_router
  type: graph-scout
  k_beam: 5                # Top-k candidate paths
  max_depth: 3             # Maximum path depth
  commit_margin: 0.15      # Confidence threshold
  cost_budget_tokens: 1000 # Token budget limit
  latency_budget_ms: 2000  # Latency budget limit
  safety_threshold: 0.2    # Lower is safer (0.0-1.0)
  prompt: "Find the best path for: {{ input }}"
```

**Decision Types:**
- `commit_next`: High confidence single path → Execute immediately
- `shortlist`: Multiple good options → Execute all sequentially  
- `no_path`: No suitable path → Fallback to response builder

**📖 [Complete GraphScout Documentation](./GRAPH_SCOUT_AGENT.md)**

### 🔘 `binary`

Returns a boolean (`"true"` or `"false"` as strings) based on a question or statement.

**Use case:** Fact checking, condition validation, flag triggering.

**Example config:**
```yaml
- id: is_fact
  type: binary
  prompt: >
    Is the following statement factually accurate? Return TRUE or FALSE.
  queue: orka:binary_check
```

### 🧾 `classification`

**⚠️ Deprecated** - kept only for backward compatibility.

This agent no longer performs classification and returns `"deprecated"`.

**Use case:** Basic topic detection (legacy support only).

### 🤖 `openai-binary`

Uses OpenAI's LLM to perform binary classification with sophisticated reasoning.

**Use case:** Complex true/false decisions requiring natural language understanding.

**Example config:**
```yaml
- id: content_appropriate
  type: openai-binary
  prompt: >
    Is this content appropriate for a professional environment?
    Content: {{ input }}
  queue: orka:moderation
```

### 🎯 `openai-classification`

Uses OpenAI's LLM to classify input into multiple predefined categories.

**Use case:** Advanced topic classification, sentiment analysis, content categorization.

**Example config:**
```yaml
- id: domain_classifier
  type: openai-classification
  prompt: >
    Classify this question into one of the following domains:
  options: [science, geography, history, technology, general]
  queue: orka:classify
```

### 📝 `openai-answer`

Builds comprehensive answers using OpenAI's LLM, typically enriched with context from previous agents.

**Use case:** Question answering, content generation, summarization.

**Example config:**
```yaml
- id: answer_builder
  type: openai-answer
  prompt: |
    Based on the search results: {{ previous_outputs.web_search }}
    And classification: {{ previous_outputs.classifier }}
    Provide a comprehensive answer to: {{ input }}
  queue: orka:answer
```

### 🏠 `local_llm`

Interfaces with locally running large language models (Ollama, LM Studio, etc.) for privacy-preserving AI processing.

**Use case:** Offline processing, privacy-sensitive applications, custom model deployment.

**Supported Providers:**
- `ollama`: Native Ollama API
- `lm_studio`: LM Studio OpenAI-compatible endpoint
- `openai_compatible`: Any OpenAI-compatible API


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcosomma/orka-reasoning](https://github.com/marcosomma/orka-reasoning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
