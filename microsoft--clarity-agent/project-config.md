---
trigger: always_on
description: This file integrates the Clarity Protocol into your AI-assisted development workflow. The Clarity Protocol helps you build systems that actually work by maintaining shared understanding of what you're building, why, and what could go wrong.
---

# Clarity Protocol Agent

This file integrates the Clarity Protocol into your AI-assisted development workflow. The Clarity Protocol helps you build systems that actually work by maintaining shared understanding of what you're building, why, and what could go wrong.

## What This Does

When building software with AI assistance, three things commonly go wrong:

1. **The AI loses track** of what you're really trying to achieve as conversations get long
2. **Failure scenarios get overlooked** because nobody's thinking systematically about what could break
3. **Decisions get lost** once they scroll out of the chat history

The Clarity Protocol solves this by creating a `.clarity-protocol/` directory alongside your code that captures:

- **The problem** you're solving and who cares about it
- **The solution** you're building and why it's designed that way
- **The failures** that could happen and how you'll handle them
- **The decisions** you've made and when to reconsider them

Think of it as the "missing manual" for your code—the stuff you can't figure out just by reading the implementation.

## When to Use Clarity Processes

The clarity agent provides several processes that help at different stages of development. The **clarity-agent** process is the natural entry point — it assesses where the project stands and routes you to the right process.

### Not Sure What to Do Next?

**Run clarity-agent**: `.clarity-agent/processes/clarity-agent.md`

The clarity-agent process looks at the current state of the `.clarity-protocol/` directory and figures out what needs attention:

- No protocol directory? Creates the structure and routes to problem clarification.
- Problem vague or missing? Routes to problem clarification.
- Problem clear but no solution? Routes to solution brainstorming.
- Solution exists but no failure analysis? Routes to failure analysis.
- Everything solid? Reports status and asks what you'd like to work on.

You can also invoke any process directly when you know what's needed.

### Clarifying What You're Building

**Process**: `~/.clarity-agent/processes/problem-clarification.md`

This is typically the first substantive process to run. It transforms vague ideas into clear problem statements through probing conversation.

**Key strategies** (see process guide for details):

- **Handle fuzzy criteria**: When you hear "fast," "easy," "secure"—use outrageous examples to help users articulate what they really mean
- **Separate problems from solutions**: When users arrive with specific implementations, probe for the underlying need they're trying to address
- **Clarify scalable criteria**: For decision-making criteria that need to be applied consistently (by humans or AI), use the rigorous 6-step criteria clarification process
- **Detect "magic"**: Flag any step that assumes someone will "just know" what another intends without explicit communication

Updates: `goal/problem.md`, `goal/stakeholders.md`, `goal/requirements.md`

### Thinking Through What Could Go Wrong

**When starting a new feature or making significant changes:**

- Load and follow the **failure analysis process**: `~/.clarity-agent/processes/failure-analysis.md`
- This orchestrates multiple "thinker" agents that examine the system from different perspectives
- Each thinker is a specialized guide in `~/.clarity-agent/thinkers/`
- Update: `failures/` directory with identified failure modes

**When you hit an unexpected problem:**

- Check existing failures in `.clarity-protocol/failures/failures.md`
- See if this matches a known failure mode
- If it's new, run failure analysis to capture it

### Making Important Decisions

**When there are multiple ways forward:**

- Load and follow the **decision guidance process**: `~/.clarity-agent/processes/decision-guidance.md`
- Walk through criteria, options, and tradeoffs
- Record the decision and when to reconsider it
- Update: `decisions/` directory

### Designing the Solution

**When you need to translate a problem into an implementation plan:**

- Load and follow the **solution brainstorming process**: `~/.clarity-agent/processes/solution-brainstorming.md`
- Explore different approaches
- This naturally triggers failure analysis and architecture design as sub-processes
- Update: `solution/solution.md`, `solution/architecture.md`

## How to Integrate This

When a project uses the Clarity Protocol, it will have a `.clarity-protocol/config.json` that points to the clarity agent installation (typically `~/.clarity-agent/`).

### For Any Project

1. Load and run **clarity-agent** (`~/.clarity-agent/processes/clarity-agent.md`)
2. The clarity agent will:
   - Create the `.clarity-protocol/` directory if it doesn't exist
   - Assess the current state of existing protocol files
   - Recommend the most valuable next process to run
3. At natural points during development, suggest relevant processes:
   - New feature? "Let's think about what could go wrong with this feature."
   - Unexpected bug? "This might be a known failure mode. Let me check..."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/clarity-agent](https://github.com/microsoft/clarity-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
