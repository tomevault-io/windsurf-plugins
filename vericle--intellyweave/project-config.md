---
trigger: always_on
description: **Detailed documentation of the Defense, Prosecution, and Judge agents.**
---

# Courthouse Debate Agents

**Detailed documentation of the Defense, Prosecution, and Judge agents.**

## Overview

The Courthouse Debate system uses three specialized agents that work together through adversarial collaboration:

```text
Defense ←→ Prosecution
      ↘   ↙
      Judge
```

Each agent has distinct responsibilities, methods, and behaviors designed to produce balanced, well-reasoned conclusions.

---

## Defense Agent

**File**: `backend/elysia/tools/courthouse/defense_agent.py`

### Role

Supports and strengthens the initial response using available evidence.

### Visual Identity

| Property | Value |
|----------|-------|
| Color | Blue |
| Icon | Shield (PiShield) |
| Gradient | `from-blue-500/10 via-blue-400/5 to-transparent` |

### Responsibilities

1. Present arguments supporting the initial response
2. Cite evidence from available source documents
3. Address prosecution challenges directly
4. Build logical case for response validity

### DSPy Signature

```python
class DefenseArgument(dspy.Signature):
    """Construct a defense for the initial response"""
    query = dspy.InputField(desc="The original user query")
    initial_response = dspy.InputField(desc="The response to defend")
    available_sources = dspy.InputField(desc="Sources supporting the response")
    prosecution_challenge = dspy.InputField(desc="Challenge from prosecution if any")
    debate_history = dspy.InputField(desc="History of the debate")

    defense = dspy.OutputField(desc="Strong defense of the initial response")
    key_evidence = dspy.OutputField(desc="Key evidence from sources")
    reasoning = dspy.OutputField(desc="Logical reasoning for the defense")
    addresses_challenges = dspy.OutputField(desc="How this addresses prosecution's challenges")
```

### Behavior

**First Round** (no prosecution challenge yet):
- Presents initial defense of the response
- Establishes evidentiary foundation
- Anticipates potential weaknesses

**Subsequent Rounds** (responding to prosecution):
- Directly addresses each prosecution challenge
- Provides counter-evidence from sources
- Strengthens weak points identified

### Output Example

```typescript
{
  agent_role: "defense",
  argument: "The initial response represents a **prudent and evidence-based approach**...\n\n**Why this methodology is appropriate:**\n\n1. Claims about deliberate exploitation require concrete evidence...",
  supporting_sources: [
    {
      title: "Paul_Stangl_85.txt",
      excerpt: "Brazilian Consular Qualification Card documentation",
      relevance: "Supporting evidence"
    }
  ],
  reasoning: "The defense argues that claims about systematic abuse require examining both the legal framework and the specific case...",
  debate_round: 1,
  agrees_with_consensus: null
}
```

### Implementation Details

```python
class DefenseAgent:
    def __init__(self, base_lm: dspy.LM):
        self.lm = base_lm
        self.argument_builder = dspy.ChainOfThought(DefenseArgument)

    async def defend(
        self,
        context: DebateContext,
        prosecution_challenge: CourthouseMessage = None
    ) -> CourthouseMessage:
        # Prepare sources summary (first 5 sources)
        sources_summary = self._summarize_sources(context.initial_sources)

        # Get prosecution challenge text
        challenge_text = prosecution_challenge.argument if prosecution_challenge else "No challenges yet"

        # Generate defense using LLM
        with dspy.context(lm=self.lm):
            result = self.argument_builder(
                query=context.initial_query,
                initial_response=context.initial_response,
                available_sources=sources_summary,
                prosecution_challenge=challenge_text,
                debate_history=debate_summary
            )

        return CourthouseMessage(
            agent_role=AgentRole.DEFENSE,
            argument=defense_text,
            supporting_sources=supporting_sources,
            reasoning=result.reasoning,
            debate_round=context.current_round,
            agrees_with_consensus=None
        )
```

---

## Prosecution Agent

**File**: `backend/elysia/tools/courthouse/prosecution_agent.py`

### Role

Critically evaluates responses through logical challenges and counter-arguments.

### Visual Identity

| Property | Value |
|----------|-------|
| Color | Red |
| Icon | Sword (PiSword) |
| Gradient | `from-red-500/10 via-red-400/5 to-transparent` |

### Responsibilities

1. Identify logical gaps in the defense
2. Challenge unsupported claims
3. Provide constructive improvement suggestions
4. Determine when to accept the defense (be "easily convincible")

### DSPy Signature

```python
class ProsecutionChallenge(dspy.Signature):
    """Construct logical challenges to the response"""
    query = dspy.InputField(desc="The original user query")
    initial_response = dspy.InputField(desc="The response to challenge")
    defense_argument = dspy.InputField(desc="Defense's latest argument")
    debate_history = dspy.InputField(desc="History of the debate")
    previous_challenges = dspy.InputField(desc="Previous challenges made")

    challenge = dspy.OutputField(desc="Logical challenge to the response or defense")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vericle/intellyweave](https://github.com/vericle/intellyweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
