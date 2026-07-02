---
trigger: always_on
description: > Template for AI-DLC projects seeded from ContextEng. Architecture is
---

# CLAUDE.md

> Template for AI-DLC projects seeded from ContextEng. Architecture is
> **AgentCore-first**: if the product has an agent/LLM loop, that loop runs on
> Amazon Bedrock AgentCore from t=0 — not as a Lambda behind API Gateway you
> migrate later. AgentCore's value is *infrastructure you delete*, not features you
> add. Two prerequisite decisions before designing the backend — the
> **Harness-vs-Runtime** fork and the **deterministic-first** gate — plus the
> primitive ledger and migration gotchas are in
> [`docs/AGENTCORE_FIRST.md`](docs/AGENTCORE_FIRST.md). Read it first.
>
> **Built for teams, not standalone developers.** These products are built and
> operated by teams inside an org, so default to shared, governed, discoverable
> infrastructure: immutable versions + named endpoints + rollback (so concurrent
> developers and deploys don't break each other), one source of truth per domain
> taxonomy (never a per-developer copy), Policy as the shared guardrail, and
> **Registry** to publish/discover agents, tools, and MCP servers across teams
> instead of each team rebuilding what already exists.

## Constraints

Never do these things:

- Never propose or implement workarounds, shortcuts, or stopgap fixes that introduce tech debt. Diagnose the root cause and design a lasting solution. When a lasting fix is not feasible now (cost, scope, upstream dependency), surface that constraint explicitly so the trade-off is a deliberate decision — not a quietly-shipped band-aid.
- **Agent code lives in the AgentCore Runtime (or Harness), not in Lambda.** The agent path is Frontend → AgentCore Runtime (direct SigV4) → Bedrock + Memory + Gateway + Identity + Policy. Building the agent loop as a Lambda behind API Gateway hits the 29-second sync timeout and forces a `turnStatus="generating"` self-invoke + frontend-polling state machine — a permanent class of async-vs-state races that AgentCore exists to delete. Runtime sessions run up to 8 hours and stream progress.
- **Deterministic-first: compute the answer when you can, generate it only when you must.** The model is the slowest, costliest, least reproducible component — reach for it last. Token cost, not model quality, is what kills agents in production (multi-agent loops use ~15× chat tokens; full history is re-sent every turn, so real loops cost 5–10× the naïve estimate). Route each requirement down three rungs: (1) plain code computes it (lookup, rule, arithmetic, validation, join) → CRUD path, no model call; (2) needs judgment in one bounded spot → a single structured / function-calling model call inside deterministic Python (router, classifier, extractor, judge); (3) genuinely un-hardcodable but verifiable → an agent loop on the Runtime. "AgentCore from t=0" means rung 3 already runs on AgentCore — *not* that every feature starts at rung 3.
- Never add a FastAPI or Express **proxy layer**. CRUD path is Frontend → API Gateway (REST) → Lambda. The agent path calls AWS-managed serverless agent services (AgentCore Runtime) directly from the frontend via Cognito Identity Pool SigV4 — that is *not* a proxy layer. Two compute paths, no custom proxy fleet. This rule is written around the principle ("no custom proxy layers in front of managed services"), not the literal ("everything goes through API Gateway"); a literal ages badly — see `docs/AGENTCORE_FIRST.md` §5.
- Never ship Strands (or any agent SDK) as a Lambda Layer. It ships inside the Runtime container's own `pyproject.toml`. SDK bumps are a one-line edit + redeploy, no layer rebuild.
- Never LLM-drive routing or orchestration. Deterministic Python decides *when* to call the model; the LLM adds wisdom inside clearly-bounded helpers. The orchestrator is plumbing, not an agent.
- Never hardcode mock data in source code. Test/mock data is acceptable only when loaded from a data source (fixtures, seed files, test APIs).
- Never deploy outside your designated region.
- Never place .md files in the root folder. All documentation goes in docs/.
- Never commit code unless the user explicitly asks.
- Never add code comments unless the user explicitly asks.
- Never call python or pip directly. Use `uv run` for execution, `uv pip` or `uv add` for package management. uv is a prerequisite.

## Harness vs Runtime — the first architectural decision

Two ways to run the loop. Decide deliberately before anything else; both are GA.

- **Runtime (code-based loop) — ContextEng's default.** You write the orchestration loop in Python (entrypoint + op-dispatch); invoke via `InvokeAgentRuntime` (SigV4); any framework (Strands, LangGraph, CrewAI, LlamaIndex, Google ADK, OpenAI Agents, or raw custom). Buys an auditable state object, in-loop gating, prompt-cache control, speculation skips, and status short-circuits. The "never LLM-drive orchestration" rule *is* a code-based-loop choice — when the orchestration doctrine is the product, write the loop.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skrinak/ContextEng](https://github.com/skrinak/ContextEng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
