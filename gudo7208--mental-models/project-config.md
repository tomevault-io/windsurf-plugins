---
trigger: always_on
description: |
---


# Multi-Disciplinary Mental Models · Munger's Lattice

> "You must know the important models from the important disciplines and use them routinely — all of them, not just a few."
> — Charlie Munger

## Usage

```
/mental-models <problem description>
```

Natural language triggers: "help me analyze from multiple angles", "use Munger's mental models", "brainstorm this"

## Architecture: Layered Loading + Parallel Analysis + Adversarial + Auto-Synthesis

Layered context loading avoids dumping all 74 models at once:

```
L0 Index (models/L0-index.md)     ← Always loaded, lightweight routing (with relevance scores)
    ↓ Pick top 4-5 disciplines (avoid going too broad)
L2 Details (models/{discipline}.md) ← Load only matched discipline files on demand
    ↓ Select 5-7 specific models (deduplicated, complementary)
Agent Team Round 1: Parallel Analysis  ← One agent per model, independent deep analysis
    ↓ Identify contradictions
Agent Team Round 2: Adversarial (optional) ← Contradicting sides respond to each other
    ↓ Synthesize
Synthesis Agent Auto-Summary       ← Cross-validation + contradiction identification + blind spot check
    ↓ Layered output
Decision Summary (3 sentences) + Full Analysis
```

## Execution Flow

### Step 1: Understand the Problem & Gather Context

1. Clarify the user's core question, decision context, and constraints
2. If the user's workspace contains relevant notes, search for background:
   ```bash
   grep -rl --include="*.md" "keyword" . | head -10
   ```
3. Read relevant notes to extract decision context
4. Use WebSearch for external information if needed

### Step 2: L0 Routing — Match Relevant Disciplines (with scoring)

Read `models/L0-index.md`, match disciplines based on problem content:
- Keywords in the problem hit discipline scenario keywords
- Problem nature directly relates to discipline's model list
- Score each matched discipline (1-5), take top 4-5
- Prefer depth over breadth: better to go deep in fewer disciplines than shallow across many

### Step 3: L2 On-Demand Loading — Select Specific Models

Only read matched discipline L2 detail files (`models/{discipline}.md`), select the most relevant models.

Selection principles:
- 5-7 models (simple problems 3-4, complex problems max 7), avoid redundancy
- Must be cross-disciplinary (at least 3 different disciplines), avoid single-perspective blind spots
- Must include at least one contrarian model (Inversion / Pre-mortem / Confirmation Bias / Survivorship Bias / Second-Order Effects)
- Prefer models with "Key Question" directly applicable to the current problem
- Deduplication: if two models would reach highly similar conclusions, keep only the more insightful one

### Step 4: Agent Team Round 1 — Parallel Analysis

Launch an independent Agent for each selected model (using Agent tool), parallel deep analysis.

Each Agent's prompt template:
```
You are an analyst specializing in "{Model Name}" ({Discipline}).

Background:
{Background gathered from workspace and search}

Problem:
{User's problem}

Model description:
{Model details loaded from L2 file}

Analyze this problem from this model's perspective:
1. What unique insight does this model provide? (Don't repeat common-sense conclusions — only output what this model uniquely offers)
2. From this model's perspective, what is the answer to the Key Question?
3. What conclusion or recommendation? (Must be specific and actionable)
4. What are the blind spots of this perspective?

Important: You may use WebSearch to find specific data to support your analysis (e.g., historical prices, statistics).
Output should be concise and powerful, no more than 300 words.
```

Agent configuration:
- Launch all Agents in a single response (multiple Agent tool calls in one message) for parallel execution
- Give Agents WebSearch access for self-directed data gathering
- subagent_type: general-purpose

### Step 4B (Optional): Adversarial Round

After Round 1, if significant contradictions are found (e.g., Model A is bullish, Model B is bearish), launch Round 2:
- Cross-send contradicting analyses
- Each Agent responds to the opposing argument: what do you agree/disagree with? Why?
- Only trigger when contradictions are significant; skip for simple problems

Adversarial Agent prompt template:
```
You are the "{Model Name}" analyst. Your previous analysis concluded:
{Round 1 analysis result}

Now, the "{Opposing Model Name}" analyst has presented a different view:
{Opposing analysis result}

Please respond:
1. Which of their arguments do you find valid?
2. What is your core position that you still maintain? Why?
3. After considering both sides, does your conclusion need revision?

No more than 200 words.
```

### Step 5: Synthesis Agent — Auto-Summary

Feed all Agent results (including adversarial round) to a dedicated "Synthesis Analyst" Agent.

Synthesis Agent prompt template:
```
You are a synthesis decision analyst, skilled at cross-validating multiple perspectives and forming final judgments.

Problem:
{User's problem}

The following are analysis results from {N} different mental models:

{All Agent results, listed by model name}

Please synthesize:

1. **Decision Summary** (3 sentences max): Give the conclusion directly, no fluff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gudo7208/mental-models](https://github.com/gudo7208/mental-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
