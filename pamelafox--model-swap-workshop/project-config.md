---
trigger: always_on
description: This repo is for the following workshop:
---

# Instructions for coding agents

This repo is for the following workshop:

## Model Swap Workshop

Frontier labs are releasing new models constantly, and it is hard to know when "better" is better enough to justify touching a working system. On top of that, "just swap the model" often turns into real work because providers expose different APIs and different expectations around tools and structured outputs.

The model swap workshop is a hands-on bake-off across frontier LLMs. We run the same scenarios using multiple models (OpenAI, Anthropic, Kimi, and more) and compare results side by side for agentic tool use, structured outputs, and multimodal tasks.

Swapping models is not just changing a model name. In this workshop, you actually do the swaps, including moving between OpenAI-style Responses APIs and Anthropic-style Messages APIs, then see what breaks and what needs to change in prompts, tool definitions, and JSON strategies.

The workshop finishes by running a small eval suite so you can quantify tradeoffs instead of relying on vibes. We provide the Microsoft Foundry environment for access to models, no account needed.

### Outline

* Introducing our models - comparison table - Kimi, DeepSeek, Mistral, GPT-5.5, Sonnet
* Single LLM calls (with optional parameters like temperature, reasoning effort)
  * [`single_llm_letter_counting.py`](examples/single_llm_letter_counting.py) — character-level counting
  * [`single_llm_spatial_reasoning.py`](examples/single_llm_spatial_reasoning.py) — multi-step rotation tracking
  * [`single_llm_self_calibration.py`](examples/single_llm_self_calibration.py) — confidence estimation
  * [`single_llm_multi_constraint.py`](examples/single_llm_multi_constraint.py) — acrostic + word count + format
* What are the knobs we can change? Prompt, parameters
* RAG: LLM call with answer grounded in citations
    * How do we get good citations?
    * Most models: Ask for them in a certain format
      * [`rag_responses.py`](examples/rag_responses.py) — OpenAI Responses API with prompt-based citations
    * Anthropic: Use citations feature
      * [`rag_messages.py`](examples/rag_messages.py) — Anthropic Messages API with built-in citations feature
* Tool calling: Can the LLMs call tools... with the right arguments?
  * [`function_calling.py`](examples/function_calling.py) — single tool, calendar event with format normalization
  * [`tool_loop_code.py`](examples/tool_loop_code.py) — code execution tool (Monty sandbox)
* Tool call selection from multiple tools
  * [`function_calling_loop.py`](examples/function_calling_loop.py) — weather + movies tools, multi-turn loop
* Image/multimodal input
  * [`image_input.py`](examples/image_input.py) — vision capabilities across models
* Agent loops: How do models handle repeated tool calls over time?
  * [`agent_trip_planner_pydanticai.py`](examples/agent_trip_planner_pydanticai.py) — PydanticAI trip planner with budget constraints
  * [`agent_trip_planner_langchain.py`](examples/agent_trip_planner_langchain.py) — LangChain trip planner
  * [`agent_trip_planner_maf.py`](examples/agent_trip_planner_maf.py) — Microsoft Agent Framework trip planner
* Evaluations: Quantify tradeoffs instead of vibes
  * [`evals_basic.py`](examples/evals_basic.py) — programmatic checks on tool calling (5 edge cases including Spanish)
  * [`evals_llm_judge.py`](examples/evals_llm_judge.py) — LLM-as-judge for RAG groundedness (direct LLM call, binary pass/fail)
  * [`evals_foundry_judge.py`](examples/evals_foundry_judge.py) — same evaluation using azure-ai-evaluation SDK (GroundednessEvaluator)
  * [`evals_agent.py`](examples/evals_agent.py) — agent eval with ToolCallAccuracyEvaluator
  * [`evals_foundry_project.py`](examples/evals_foundry_project.py) — openai.evals.create via Foundry project (optional, requires project)

## Samples in this repo

All examples authenticate to Foundry using an API key and reference environment variables from a `.env` file.

**Note:** `structured_outputs.py` is kept for reference but removed from the workshop flow. Rationale:
1. Structured outputs require 3 different approaches across models (responses.parse for GPT, tool-calling fallback for others, Messages API for Anthropic), making it awkward to present as a single example.
2. Tool calling already demonstrates structured output extraction — models must produce valid JSON matching a schema.
3. Removing it gives a better flow from tool calling straight into agentic patterns.

Key SDKs/frameworks used:
- **OpenAI Python SDK** (`openai`): For calling Foundry-hosted OpenAI models via the Responses API.
- **Anthropic Python SDK** (`anthropic`): For calling Foundry-hosted Claude models via the Messages API.
- **PydanticAI** (`pydantic-ai`): Agent framework with typed tool support, works with OpenAI or Anthropic providers.
- **LangChain** (`langchain`, `langchain-azure-ai`): Agent framework using `AzureAIOpenAIApiChatModel` for OpenAI via Responses API.
- **Microsoft Agent Framework** (`agent-framework-*`): Microsoft's agent framework, supporting OpenAI and Anthropic clients.
- **DSPy** (`dspy`): Automated prompt optimization framework.

Some files not in the main workshop flow use additional packages via PEP 723 inline script metadata:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pamelafox/model-swap-workshop](https://github.com/pamelafox/model-swap-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
