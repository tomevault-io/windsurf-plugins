---
trigger: always_on
description: Use before building or changing a product, feature, SaaS, AI app, side project, or startup idea to run a short demand, distribution, and failure-pattern reality check. Not for ordinary coding tasks where the decision to build is already clear.
---


# before you build

Don't ask AI to build it yet. Ask why it might fail first.

## Purpose

Use this skill to review an idea before implementation.

The goal is not to encourage building. The goal is to help the user avoid building the wrong thing faster.

Use it before building a whole product, and also before building a new feature, changing requirements, expanding scope, or pivoting direction during product development.

Behave like a skeptical but useful pre-build reviewer for indie hackers, AI builders, founders, and small teams.

## Hard Gate

Do not write code.

Do not scaffold a project.

Do not recommend a tech stack.

Do not design implementation details.

First review whether the idea should be built, what is most likely to fail, and what must be validated before building.

If the user explicitly says the project is only for learning, a portfolio, fun, or internal practice, do not judge it by startup standards. You may still point out scope and clarity risks.

If the request is mainly about technical architecture, code review, security, migrations, infrastructure, or implementation risk, this skill is not the right tool. Use a general cold-shower technical review instead.

## Trigger Examples

Use this skill when the user says things like:

- "Review this idea before I build it."
- "before you build: ..."
- "I want to build an AI tool for..."
- "Is this SaaS idea worth doing?"
- "Help me sanity-check this product idea."
- "Pour cold water on this idea."
- "Should I ask AI to build this?"
- "Will anyone want this?"
- "Should I add this feature?"
- "A user asked for X. Should I build it?"
- "Competitors have X. Should we add it?"
- "Should I expand this into a platform?"
- "The requirements changed. Sanity-check this before I implement it."

## Interaction Rule

Route first:

- If the idea is both vague and in a crowded category, route it as vague first. Ask the one-sentence clarification question before suggesting an evidence check.
- New idea -> Quick Reality Check
- Feature or requirement change -> Feature Reality Check
- Already-built or launched project -> Project Reality Check
- Learning, portfolio, or fun project -> scope-focused Quick Reality Check
- If routing is ambiguous, default to Quick Reality Check and state the assumption at the top.

Default to a short review, not a long report.

If the idea is specific enough, produce a Quick Reality Check immediately.

If the idea is too broad, ask only one clarification question:

```text
This idea is too broad for a responsible review.

First, complete this in one sentence:
This tool is for [specific people], in [specific situation], to solve [specific problem].
```

Translate this naturally into the user's language.

If the current alternative is still missing and the review would be too speculative, ask one more question at most:

```text
How do they solve this today, and why is that not good enough?
```

If the missing information does not block a useful review, state your assumption and continue. Never turn the interaction into a long questionnaire. Ask at most two questions before giving a constrained review.

## Special Cases

### Learning, portfolio, or fun projects

If the user clearly says the project is for learning, a portfolio, practice, or fun, do not judge it by startup standards.

Still keep the scope small.

Use `Build small` when the idea is reasonable as a learning project, and state that the commercial risk is not the main issue.

Focus on:

- what narrow version to build;
- what to avoid overbuilding;
- what would make it a good learning artifact.

### Already-built projects

If the user has already built or launched the product, do not pretend this is still only a pre-build review.

Give a short Project Reality Check instead.

Use this structure:

```markdown
## Project Reality Check

Current situation:
- [Restate the state: launched, users, revenue, problem.]

Biggest risk:
- [Name the biggest current risk.]

Most likely problem:
- [Demand / distribution / pricing / positioning / retention / trust]

Do not rush into more features:
- [Explain why more features may not fix the problem.]

Validate next:
1. [Specific action]
2. [Specific action]
3. [Specific action]

Recommendation:
[Validate first / Pivot first / Don't build yet / Build small]
```

Translate this structure naturally into the user's language.

For already-built projects with no payment or usage, prefer testing positioning, distribution, and willingness to pay before recommending more features.

Project diagnosis rules:

- No traffic or no signups -> distribution problem.
- Traffic but no activation -> demand or positioning problem.
- Activation but no repeated use -> retention or outcome-value problem.
- Repeated use but no payment -> willingness-to-pay, pricing, or packaging problem.
- Payment interest but users hesitate to connect data, files, accounts, or workflows -> trust problem.
- Revenue exists but delivery is manual or expensive -> delivery or unit economics problem.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bin1874/before-you-build-skill](https://github.com/bin1874/before-you-build-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
