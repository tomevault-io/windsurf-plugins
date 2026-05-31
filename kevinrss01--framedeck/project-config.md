---
trigger: always_on
description: Based on https://developers.openai.com/api/docs/guides/prompt-guidance
---


Based on https://developers.openai.com/api/docs/guides/prompt-guidance

# GPT-5.5 prompting guide

Prompt GPT-5.5 with outcome-first goals, concise style controls, retrieval budgets, and validation loops.

## New in GPT-5.5 vs GPT-5.4
- Shorter, outcome-first prompts usually work better than process-heavy prompt stacks.
- More efficient reasoning means `low` and `medium` effort should be re-evaluated before escalating.
- Preambles, `phase` handling, and assistant-item replay remain important for tool-heavy Responses workflows.
- Explicit personality, retrieval budgets, and validation rules help shape customer-facing and agentic UX.

GPT-5.5 works best when prompts define the outcome and leave room for the model to choose an efficient solution path. Compared with earlier models, you can often use shorter, more outcome-oriented prompts: describe what good looks like, what constraints matter, what evidence is available, and what the final answer should contain.

Avoid carrying over every instruction from an older prompt stack. Legacy prompts often over-specify the process because earlier models needed more help staying on track. With GPT-5.5, that can add noise, narrow the model's search space, or lead to overly mechanical answers.

For more detail on GPT-5.5 behavior changes, start with the [Using GPT-5.5 guide](https://developers.openai.com/api/docs/guides/latest-model). This guide focuses on prompt changes that follow from those behavior changes.

The patterns here are starting points. Adapt them to your product surface, tools, evals, and user experience goals.

## Automated migration with Codex

Codex can implement the changes from this guide with the [OpenAI Docs Skill](https://github.com/openai/skills/tree/main/skills/.curated/openai-docs).

```text
$openai-docs migrate this project to gpt-5.5
```

To use this skill in other coding agents, download it from the [OpenAI skills repository](https://github.com/openai/skills/tree/main/skills/.curated/openai-docs).

## Personality and behavior

GPT-5.5's default style is efficient, direct, and task-oriented. This is useful for production systems: responses stay focused, behavior is easier to steer, and the model avoids unnecessary conversational padding.

For customer-facing assistants, support workflows, coaching experiences, and other conversational products, define both personality and collaboration style.

- **Personality** controls how the assistant sounds: tone, warmth, directness, formality, humor, empathy, and level of polish.
- **Collaboration style** controls how the assistant works: when it asks questions, when it makes assumptions, how proactive it should be, how much context it gives, when it checks work, and how it handles uncertainty or risk.

Keep both short. Personality instructions should shape the user experience. Collaboration instructions should shape task behavior. Neither should replace clear goals, success criteria, tool rules, or stopping conditions.

Example personality block for a steady task-focused assistant:

```text
# Personality
You are a capable collaborator: approachable, steady, and direct. Assume the user is competent and acting in good faith, and respond with patience, respect, and practical helpfulness.

Prefer making progress over stopping for clarification when the request is already clear enough to attempt. Use context and reasonable assumptions to move forward. Ask for clarification only when the missing information would materially change the answer or create meaningful risk, and keep any question narrow.

Stay concise without becoming curt. Give enough context for the user to understand and trust the answer, then stop. Use examples, comparisons, or simple analogies when they make the point easier to grasp. When correcting the user or disagreeing, be candid but constructive. When an error is pointed out, acknowledge it plainly and focus on fixing it.

Match the user's tone within professional bounds. Avoid emojis and profanity by default, unless the user explicitly asks for that style or has clearly established it as appropriate for the conversation.
```

Example personality block for an expressive collaborative assistant:

```text
# Personality
Adopt a vivid conversational presence: intelligent, curious, playful when appropriate, and attentive to the user's thinking. Ask good questions when the problem is blurry, then become decisive once there is enough context.

Be warm, collaborative, and polished. Conversation should feel easy and alive, but not chatty for its own sake. Offer a real point of view rather than merely mirroring the user, while staying responsive to their goals and constraints.

Be thoughtful and grounded when the task calls for synthesis or advice. State a clear recommendation when you have enough context, explain important tradeoffs, and name uncertainty without becoming evasive.
```

For more expressive products, add warmth, curiosity, humor, or point of view explicitly, but keep the block short. Use personality to shape the experience, not to compensate for unclear goals or missing task instructions.

## Improve time to first visible token with a preamble


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinrss01/framedeck](https://github.com/kevinrss01/framedeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
