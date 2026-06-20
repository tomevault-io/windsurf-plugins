---
trigger: always_on
description: AI-augmented software engineering workflow synthesized from Addy Osmani's methodology, Anthropic's agentic coding research, and 30+ real-world projects. Supports two collaboration modes — interactive pair programming and autonomous delegation — with structured planning, context management, strategic model selection, human accountability, and continuous learning.
---


# LLM Coding Workflow

AI-augmented software engineering workflow that maximizes LLM effectiveness through structured planning, clear context, strategic delegation, and human accountability.

## What This Skill Does

This skill synthesizes Addy Osmani's LLM coding workflow, Anthropic's agentic coding research, Harper Reed's spec-driven pipeline, and patterns from 30+ real-world projects. It supports two collaboration modes — interactive pair programming and autonomous delegation — and provides a systematic approach to AI-assisted development that prevents wasted cycles while enabling increasingly autonomous execution.

**Core Philosophy**: The human engineer is the accountable owner; the AI is a capable collaborator whose autonomy scales with the quality of the prompt. Well-scoped delegation with clear acceptance criteria is not "blind trust" — it is a higher-leverage operating mode.

**Two Collaboration Modes:**

| Mode | When | How | Review |
|------|------|-----|--------|
| **Pair Mode** (Conductor) | Ambiguous problems, design decisions, learning new domains | Interactive back-and-forth; AI as thought partner | Line-by-line as you go |
| **Delegation Mode** (Orchestrator) | Well-scoped tasks with clear acceptance criteria | Structured prompt → autonomous execution → human review | Deliverable review against spec |

**Key Capabilities:**
- **Structured Planning**: Rapid waterfall-style planning before implementation
- **Chunked Execution**: Small, focused tasks with quick course correction
- **Context Management**: Systematic context provision for optimal AI output
- **Model Selection**: Strategic switching between models based on task needs
- **Human Accountability**: You own the output; review depth scales with delegation scope
- **Delegation Prompts**: Spec-driven handoff for autonomous execution
- **Granular Commits**: Frequent save points for easy rollback
- **Continuous Learning**: Skill amplification through AI collaboration

## Prerequisites

**Required:**
- Git repository initialized
- Testing framework configured
- Linter/formatter installed

**Recommended:**
- CI/CD pipeline configured
- Type checker enabled (TypeScript, mypy, etc.)
- Context packaging tool (gitingest, repo2txt)

## Quick Start

### Begin a New Feature
```bash
# Step 1: Plan with AI
/llm-workflow plan "Implement user authentication with JWT"

# Step 2: Execute in chunks
/llm-workflow implement --chunk-size small

# Step 3: Review and commit
/llm-workflow review --commit
```

### Debug an Issue
```bash
/llm-workflow debug "Memory leak in event handlers"
```

### Refactor Code
```bash
/llm-workflow refactor src/legacy/utils.js --modernize
```

---

## Complete Guide

### The 10 Principles

This workflow is built on 10 core principles that maximize AI effectiveness:

#### 0. Interview Before Planning

**Before you plan, make sure you're planning the right thing.** Most people show up with a solution in their head ("build me a dashboard") when the real need is something different ("I need three numbers emailed to me every Monday"). This one prompt closes the gap:

> *"I'm about to start this project. Interview me until you have 95% confidence about what I actually want, not what I think I should want."*

This flips the dynamic. Instead of you pitching an idea and hoping the AI reads your mind, the AI asks the questions that surface assumptions you did not know you were making. It separates the _what_ from the _why_ before a single line of planning gets done.

**How to use it:**
- Start every new project or feature with this prompt (or something like it)
- Answer honestly, including "I don't know yet" — the gaps are where the value is
- Let the AI play it back: "So what you actually need is ___. Is that right?"
- Only after confirmation should you move to Principle 1 (planning)

This is especially valuable in **Delegation Mode** — if the spec is wrong, everything downstream is wrong. Five minutes of interviewing saves five hours of rework.

#### 1. Plan Before Coding

**Never start implementation without a plan.** Use AI to rapidly create detailed specifications.

```
You: I need to implement [feature]. Help me create a detailed spec including:
- Functional requirements
- Edge cases
- Technical constraints
- Preferred patterns
- Success criteria
```

**Workflow:**
```bash
# Create comprehensive plan
/llm-workflow plan "[feature description]"

# Output includes:
# - Requirements breakdown
# - Task decomposition
# - Risk assessment
# - Testing strategy
```

This is "waterfall in 15 minutes" - rapid structured planning that prevents wasted cycles.

#### 2. Break Work Into Small Chunks

**Never request monolithic outputs.** Feed the LLM focused, manageable tasks one at a time.

**Bad:**
```
Write a complete authentication system with login, logout,
password reset, 2FA, session management, and user profiles.
```

**Good:**
```
Task 1: Create the login endpoint with basic validation
Task 2: Add password hashing utility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericporres/llm-coding-workflow-skill](https://github.com/ericporres/llm-coding-workflow-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
