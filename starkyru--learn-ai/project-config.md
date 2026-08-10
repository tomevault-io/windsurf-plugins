---
trigger: always_on
description: This file is auto-loaded as context. Read it, then help the learner well.
---

# AGENTS.md — context for agents working in `learn-ai`

This file is auto-loaded as context. Read it, then help the learner well.

## What this is

`learn-ai` is a **personal, hands-on AI course** — a project-based curriculum that
takes one person from a vague sense of how AI apps work to building real ones:
provider integration, embeddings, RAG, agents, multimodal apps, fine-tuning,
LLMOps/eval, security, and AI product UX, in **both TypeScript and Python**. The
reader is the _learner_; you are their tutor/pair-programmer. Favor teaching over
just shipping code (see `/tutor` below).

## Repo layout

```text
learn-ai/
├── packages/
│   ├── ts/llm-core/        # provider-agnostic LLM client (TypeScript) — @learn-ai/llm-core
│   └── py/llm_core/        # the same client, in Python — llm_core
├── modules/
│   ├── 00-setup/ … 23-capstone/   # main numbered modules
│   ├── 01b-*/ 05b-*/ 06b-*/ 06c-*/ 13b-*  # companion/deep-dive modules
│   └── <module>/
│       ├── README.md       # THE LESSON: concepts + numbered tasks + "Done when" checklist
│       ├── ts/             # TypeScript exercises
│       └── py/             # Python exercises
├── projects/
│   ├── news-agent/         # applied: a Telegram daily-news agent (build after module 06)
│   └── tutor/              # standalone Python study CLI: `uv run python -m tutor ...`
├── data/                   # sample corpora for RAG exercises
├── scripts/                # helpers (smoke tests, etc.)
├── docs/                   # cross-cutting docs (see docs/TUTOR_AND_EXAM.md)
└── .claude/commands/       # /tutor and /exam slash commands
```

The module map:

```text
00 Setup & Providers
01 LLM Fundamentals
01b Classic ML Foundations (companion — extends 01; from-scratch numpy/TS, no provider)
01c Deep Learning Essentials (companion — extends 01; from-scratch numpy/TS, no provider)
01d Transformer Architecture (companion — extends 01; from-scratch numpy/TS, no provider)
01e Trees & Ensembles (companion — extends 01; from-scratch numpy/TS, no provider)
01f Stats Foundations (companion — extends 01; from-scratch numpy/TS, no provider)
02 LLM Integration
03 Prompting & Patterns
04 Embeddings & Vectors
05 RAG
05b Advanced RAG (deep dive — extends 05; see docs/ADVANCED_RAG.md)
06 Agents
06b LangGraph (deep dive — extends 06; see docs/LANGGRAPH.md)
06c Agent Frameworks (companion — LangChain/CrewAI/AutoGen, extends 06/06b; offline via --stub)
07 Advanced & Production
08 Classification
09 Computer Vision
10 Image Generation
11 Document Ingestion
12 Text-to-SQL
13 Fine-tuning
13b Alignment & Post-training (deep dive — extends 13; offline toy RLHF/DPO)
14 Local Inference & Optimization
15 Reasoning & Test-time Compute
16 Context Engineering
17 MCP & Modern Agent APIs
18 Computer Use
19 Audio & Speech
20 AI Security
21 LLMOps & Eval
22 AI Product UX
23 Capstone
```

**Each module's `README.md` is the source of truth** for what the learner is
studying — read it before tutoring, reviewing, or quizzing on a topic. If a
module README is missing or incomplete, fall back to the root `README.md` /
`CURRICULUM.md` and say so rather than inventing content.

## Depth-level convention (🟢 / 🟡 / 🔴)

Modules use three depth lanes where applicable; the learner picks a lane per
module/task:

- 🟢 **App** — build something that works using the ecosystem/libraries.
- 🟡 **Balanced** — build the app _and_ implement one core piece by hand for intuition.
- 🔴 **Deep** — implement the machinery from scratch (tokenizer, attention,
  vector index, ReAct loop, KV cache, LoRA update, etc.). 🔴 tasks often
  **forbid the obvious library** (no `tiktoken` for the tokenizer, no `chromadb`
  for the vector index) — that constraint is the point. Don't suggest a banned
  library for a 🔴 task.

Respect the learner's chosen lane when advising or reviewing.

## Companion and deep-dive modules

Several modules extend the main 00-23 path. Treat them as first-class lessons:

- `01b` through `01f` are offline foundations companions: classic ML, deep
  learning, transformers, trees/ensembles, and statistics. They should not use a
  provider unless the local README explicitly says otherwise.
- `05b-advanced-rag` covers named RAG architectures: Contextual Retrieval,
  Corrective RAG (CRAG), Self-RAG, and GraphRAG. Use
  `docs/ADVANCED_RAG.md` as the cross-cutting reference.
- `06b-langgraph` is the LangGraph production deep dive; use
  `docs/LANGGRAPH.md` as the cross-cutting reference.
- `06c-agent-frameworks` maps course-built patterns to LangChain, CrewAI,
  AutoGen, LlamaIndex, and Semantic Kernel. Exercises can run offline with
  `--stub`.
- `13b-alignment` covers post-training and alignment from scratch on toy models:
  preference data, Elo/win rates, Bradley-Terry reward modeling, RLHF with a
  reward-hacking demo, KL control, and DPO.

## The `llm_core` abstraction (default rule)

Exercise code that needs a text/chat/embedding model goes through the shared
client by default:

- Python: `from llm_core import get_provider, ChatMessage, ChatOptions` →
  `get_provider(name=None)` reads `LLM_PROVIDER` (default `ollama`); `provider.chat(...)`,
  `provider.chat_stream(...)`, `provider.embed(...)`.
- TypeScript: `getProvider()` from `@learn-ai/llm-core`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starkyru/learn-ai](https://github.com/starkyru/learn-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
