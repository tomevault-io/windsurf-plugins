---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Copilot, Codex, Windsurf, etc.) working
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, Codex, Windsurf, etc.) working
**on** the Agent Kernel codebase itself. If you're building an agent *with* Agent Kernel, see
the [README](README.md) and [kernel.yaala.ai/docs](https://kernel.yaala.ai/docs) instead — this
file is for contributors to this repo.

## Two skill sets — don't confuse them

This repo ships two unrelated sets of "skills," named similarly but serving opposite audiences:

| Skill set | Location | Audience | Purpose |
| --- | --- | --- | --- |
| **Dev skills** | [.agents/skills/ak-dev-*](.agents/skills/) | You, working on this repo | Architecture, testing conventions, code quality, and step-by-step guides for adding a new framework adapter / guardrail / knowledge base / messaging integration / tracing provider / multimodal storage / sandbox provider / queue transport to Agent Kernel core |
| **Bundled skills** | [ak-py/src/agentkernel/skills/](ak-py/src/agentkernel/skills/) (`ak-init`, `ak-build`, `ak-add-capabilities`, `ak-add-integration`, `ak-cloud-deploy`, `ak-test`) | End users of the `agentkernel` PyPI package | Shipped *inside* the package so a downstream developer's coding assistant can scaffold and extend agents built *with* Agent Kernel |

If a task is about changing Agent Kernel's own source, use the dev skills under `.agents/skills/`.
Never edit the bundled skills to document a dev-only workflow, and never edit the dev skills to
change end-user-facing scaffolding behavior — read the description in each `SKILL.md` before
editing either.

**Before non-trivial changes to core**, load `.agents/skills/ak-dev-architecture` — it covers
`Session`, `Agent`, `Runner`, `Module`, `Runtime`, `AgentService`, `ChatService` (and which layer
each surface calls), `AKConfig`, hooks, tools, multimodal, conversation threads, and the ECS
containerized deployment classes in depth. Don't re-derive this from
scratch by grepping; the skill is maintained precisely so agents don't have to.

## Repo map

```
ak-py/                  The agentkernel PyPI package (core framework, all Python source + tests)
  src/agentkernel/
    core/                Framework-agnostic abstractions (Session, Agent, Runner, Module, Runtime, Config, hooks, tools)
    framework/           Adapters: openai, crewai, langgraph, adk, smolagents
    api/                 REST, MCP, A2A server layers
    deployment/          AWS (Lambda + ECS), Azure Functions, GCP Cloud Run handlers
    integration/         adapter/ (the messaging seam) + Slack, WhatsApp, Messenger, Instagram, Telegram, Teams, Gmail
    knowledgebase/       ChromaDB, Neo4j, Starburst backends
    guardrail/           OpenAI, AWS Bedrock, Walled AI guardrail providers
    trace/               Langfuse, OpenLLMetry tracing adapters
    skills/              Bundled end-user skills (see table above) — not dev docs
  tests/                 pytest suite, mirrors src/ structure
ak-deployment/           Terraform modules per cloud (aws / azure / gcp) x (serverless / containerized)
examples/                Runnable sample apps per framework/deployment combo
use-cases/               End-to-end agents built from a SPEC.md using the bundled skills
e2e/                     Messaging integration e2e harness: deployable app + Terraform + pytest suite driven against real platform accounts (see e2e/README.md)
docs/                    Docusaurus site (kernel.yaala.ai/docs) — versioned_docs/ are frozen, don't edit old versions
.agents/skills/          Dev skills (see table above)
```

The **core never depends on** `framework/`, `integration/`, `deployment/`, or `api/`. If a change
to `core/` requires importing something from those, that's a design smell — stop and reconsider,
or check with the maintainer.

## Adapter architecture — stay unopinionated

Every integration point (framework adapter, guardrail provider, knowledge base backend, session
store, messaging integration, tracing provider) is a **thin adapter behind a stable core
interface**, not a rewrite or reinterpretation of the underlying tool. The core abstraction
(`Agent`, `Runner`, `Module`, `Tool`, `KnowledgeBase`, `SessionStore`, `BaseTrace`, ...) defines the
minimal contract; the adapter's job is to wrap the native object/API as-is and translate at the
boundary — nothing more.

When adding or touching an adapter:

- **Wrap, don't abstract over.** Expose the native framework/service object with minimal
  reshaping. Don't invent a new intermediate abstraction "for consistency" across adapters —
  each adapter can look different internally if that's what its underlying tool naturally wants;
  forcing uniformity across adapters is itself an opinion the architecture avoids.
- **No feature-forcing.** Don't require a capability the underlying framework/service doesn't
  natively support (e.g. don't invent a fake token stream when the SDK has no streaming API —
  raise `NotImplementedError` until a real mapping exists). CrewAI and smolagents *do* stream in
  their SDKs; their AK adapters still raise because that wiring is not implemented yet — same
  `NotImplementedError` shape, different reason.
- **No hidden defaults that change behavior.** Config-driven, explicit choices (`AKConfig`) beat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaalalabs/agent-kernel](https://github.com/yaalalabs/agent-kernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
