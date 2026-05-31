---
trigger: always_on
description: Based on https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md
---


Based on https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md

# Prompting best practices

Comprehensive guide to prompt engineering techniques for Claude's latest models, covering clarity, examples, XML structuring, thinking, and agentic systems.

---

This is the single reference for prompt engineering with Claude's latest models, including Claude Opus 4.7, Claude Opus 4.6, Claude Sonnet 4.6, and Claude Haiku 4.5. It covers foundational techniques, output control, tool use, thinking, and agentic systems. Jump to the section that matches your situation.

<Tip>
  For an overview of model capabilities, see the [models overview](/docs/en/about-claude/models/overview). For details on what's new in Claude Opus 4.7, see [What's new in Claude Opus 4.7](/docs/en/about-claude/models/whats-new-claude-4-7). For migration guidance, see the [Migration guide](/docs/en/about-claude/models/migration-guide).
</Tip>

## Prompting Claude Opus 4.7

Claude Opus 4.7 is our most capable generally available model, with particular strengths in long-horizon agentic work, knowledge work, vision, and memory tasks. It performs well out of the box on existing Claude Opus 4.6 prompts. The patterns below cover the behaviors that most often require tuning.

<Note>
For API parameter changes when migrating from Claude Opus 4.6 (effort levels, task budgets, thinking configuration, sampling-parameter removal, and tokenization), see the [migration guide](/docs/en/about-claude/models/migration-guide#migrating-to-claude-opus-4-7).
</Note>

### Response length and verbosity

Claude Opus 4.7 calibrates response length to how complex it judges the task to be, rather than defaulting to a fixed verbosity. This usually means shorter answers on simple lookups and much longer ones on open-ended analysis.

If your product depends on a certain style or verbosity of output, you may need to tune your prompts. As an example, to decrease verbosity, you might add:

```text
Provide concise, focused responses. Skip non-essential context, and keep examples minimal.
```

If you see specific examples of kinds of verbosity (i.e. over-explaining), you can add additional instructions in your prompt to prevent them. Positive examples showing how Claude can communicate with the appropriate level of concision tend to be more effective than negative examples or instructions that tell the model what not to do.

### Calibrating effort and thinking depth

The [effort parameter](/docs/en/build-with-claude/effort) allows you to tune Claude's intelligence vs. token spend, trading off capability for faster speed and lower costs. Start with the new `xhigh` effort level for coding and agentic use cases, and use a minimum of `high` effort for most intelligence-sensitive use cases. Experiment with other effort levels to further tune token usage and intelligence:

- **`max`:** Max effort can deliver performance gains in some use cases, but may show diminishing returns from increased token usage. This setting can also sometimes be prone to overthinking. We recommend testing max effort for intelligence-demanding tasks.
- **`xhigh` (new):** Extra high effort is the best setting for most coding and agentic use cases.
- **`high`:** This setting balances token usage and intelligence. For most intelligence-sensitive use cases, we recommend a minimum of `high` effort.
- **`medium`:** Good for cost-sensitive use cases that need to reduce token usage while trading off intelligence.
- **`low`:** Reserve for short, scoped tasks and latency-sensitive workloads that are not intelligence-sensitive.

Meaningfully changing from Claude Opus 4.6, Claude Opus 4.7 respects effort levels strictly, especially at the low end. At `low` and `medium`, the model scopes its work to what was asked rather than going above and beyond. This is good for latency and cost, but on moderately complex tasks running at `low` effort there is some risk of under-thinking.

If you observe shallow reasoning on complex problems, raise effort to `high` or `xhigh` rather than prompting around it. If you need to keep effort at `low` for latency, add targeted guidance:

```text
This task involves multi-step reasoning. Think carefully through the problem before responding.
```

We expect effort to be more important for this model than for any prior Opus, and recommend experimenting with it actively when you upgrade.

The triggering behavior for adaptive thinking is steerable. If you find the model thinking more often than you'd like — which can happen with large or complex system prompts — add guidance to steer it. As always, measure the effect of any prompting changes on performance. Example:

```text
Thinking adds latency and should only be used when it will meaningfully improve answer quality — typically for problems that require multi-step reasoning. When in doubt, respond directly.
```

Conversely, if you're running hard workloads at `medium` and seeing under-thinking, the first lever is to raise effort. If you need finer control, prompt for it directly.

<Note>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinrss01/framedeck](https://github.com/kevinrss01/framedeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
