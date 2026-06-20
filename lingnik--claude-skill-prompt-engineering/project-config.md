---
trigger: always_on
description: Help users craft effective prompts for Claude models (Opus 4.6, Sonnet 4.6, Haiku 4.5) across API and agent harness contexts. Trigger when the user asks to write, improve, or debug a prompt, choose a Claude model, configure thinking or effort, migrate from an older model, set up CLAUDE.md, write task instructions for Claude Code or Cowork, decompose a task for an agent, or write a repeatable process prompt. Also trigger for natural phrasing like "getting bad results from Claude," "Claude keeps d
---


# Claude Prompt Engineering Skill

Help users and agents craft effective prompts for Claude models (Opus 4.6, Sonnet 4.6, Haiku 4.5) across all deployment contexts — from direct API usage to conversational agent harnesses like Claude Code, Cowork, and Chat. Guidance is grounded in system card analysis and official documentation.

## Trigger & Scope

Identify the user's context to determine which guidance applies. This skill covers Opus 4.6, Sonnet 4.6, and Haiku 4.5 across API, agent harness, and agentic system-building contexts. It does not cover older models except in the context of migration.

## Workflow

Follow these steps when helping with prompt engineering. Each step includes a skip condition — check it before proceeding.

These steps are a framework, not a rigid procedure. Adapt to the user's specific situation — skip steps that don't apply, combine steps when natural, and reason from the Key Principles when encountering scenarios these instructions don't cover explicitly.

### Step 0: Identify the Prompting Context

Determine which context the user is working in. This changes which advice and reference files apply. Heuristic: if the user mentions Claude Code, Cowork, or Chat, or is writing a user-turn message rather than a system prompt, they are in `agent_harness` context. If they are writing system prompts and tool definitions, they are in `api` context. If they are designing an agent product for end users, they are in `building_harness` context.

<routing>
  <context name="api">
    User controls: system prompt, tools, model, thinking config, all parameters.
    Primary guidance: this file + all reference files.
  </context>
  <context name="agent_harness">
    User controls: user-turn messages, CLAUDE.md, plan mode, session model choice.
    Claude Code also: system prompt customization, tool restrictions, effort/thinking control, compaction.
    Cowork/Chat: base system prompt, tool definitions, and thinking config are fixed.
    Primary guidance: references/agent-harness-prompting.md
  </context>
  <context name="building_harness">
    User controls: system prompt, tools, model — designing for end users.
    Primary guidance: references/agentic-prompting.md
  </context>
</routing>

If the user is in an **agent harness context**, prioritize guidance from `references/agent-harness-prompting.md`. The API-focused steps below (system prompt design, API parameter configuration) still apply when the user is building their own system or using the API directly.

This skill provides guidance and drafts. Do not modify the user's files (system prompts, CLAUDE.md, code) without explicit permission — present recommendations and let the user decide what to apply.

### Step 1: Understand the Goal

Before writing or modifying a prompt, clarify:

- **What is the task?** Classification, extraction, generation, analysis, coding, agentic workflow, conversation?
- **What is the input?** Short text, long documents, images, code, structured data?
- **What is the expected output?** Free text, JSON, code, tool calls, decisions?
- **What are the constraints?** Latency, cost, accuracy requirements, safety sensitivity?
- **What's the deployment context?** Single API call, multi-turn conversation, agent loop, batch processing, or agent harness (Claude Code/Cowork/Chat)?

### Step 2: Select the Model

Skip if: the user already specified a model, or is in a harness where model choice is fixed for the session.

Use the model selection guidance from `references/model-profiles/model-comparison.md` and the individual model profiles. The short version:

| If the task is... | Use | Why |
|-------------------|-----|-----|
| Complex multi-step coding or deep investigation | **Opus 4.6** | Highest capability ceiling; 128K output tokens |
| Production workload needing quality + efficiency | **Sonnet 4.6** | Best quality/cost ratio; SOTA on web agents and finance |
| High-volume, moderate-complexity, or parallel agents | **Haiku 4.5** | 5× cheaper than Opus output; fastest latency |
| Sensitive topics needing low refusals + strong capability | **Sonnet 4.6** | 0.41% over-refusal with best safety (99.40% on hard violative). Haiku is even lower (0.02%) but less capable. |
| Agentic deployment where injection is a concern | **Sonnet 4.6 + thinking + safeguards** | 0% injection success on coding benchmarks |

If unsure, start with Sonnet 4.6 at medium effort. It matches Opus on most benchmarks at 60% of the cost.

### Step 3: Draft the Prompt

Skip if: the user is in an agent harness context — they write user-turn messages, not structured prompts. Consult `references/agent-harness-prompting.md` instead.

Build the prompt using this structure:

1. **System prompt** — Role, behavioral constraints, output format rules
2. **Context/documents** — Long content goes first, above the query

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lingnik/claude-skill-prompt-engineering](https://github.com/Lingnik/claude-skill-prompt-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
