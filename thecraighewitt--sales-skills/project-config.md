---
trigger: always_on
description: **Sales Skills** is an open-source collection of 21 AI agent skills covering the full B2B sales lifecycle. Built by [Craig Hewitt](https://agentprime.io), founder of Castos ($1.5M ARR) and AgentPrime.
---

# CLAUDE.md -- Sales Skills

## Repository Overview

**Sales Skills** is an open-source collection of 21 AI agent skills covering the full B2B sales lifecycle. Built by [Craig Hewitt](https://agentprime.io), founder of Castos ($1.5M ARR) and AgentPrime.

- **GitHub:** https://github.com/thecraighewitt/sales-skills
- **License:** MIT
- **Spec:** [Agent Skills Specification](https://agentskills.io/specification.md)

## Philosophy

These skills are opinionated by design. They encode real practitioner experience -- not sales textbook theory or "it depends" hedging.

- **Push back on weak inputs.** If a user gives vague ICP definitions ("we sell to everyone"), lazy value props ("we help companies grow"), or unsubstantiated claims ("we're the best"), challenge them. Good output requires good input.
- **Frameworks over theory.** Every skill gives users a structure they can follow immediately. "Use MEDDIC for enterprise discovery" is more useful than a paragraph about why qualification matters.
- **Direct and opinionated.** "Do this, not that" beats "consider your options." Take a stance. If something doesn't work in practice, say so.
- **Examples over explanations.** Show the cold email. Show the discovery question. Show the objection response. Don't describe what one should look like.
- **Channel-specific.** Each outreach channel has its own skill with channel-specific best practices. Don't bleed LinkedIn advice into the cold email skill or vice versa.

## Repository Structure

```
sales-skills/
├── .claude-plugin/marketplace.json   # Claude Code plugin manifest
├── skills/                           # All 21 skills
│   └── skill-name/SKILL.md          # One SKILL.md per directory
├── CLAUDE.md                         # This file (agent instructions)
├── CONTRIBUTING.md                   # How to contribute
├── VERSIONS.md                       # Version tracking
├── README.md                         # Public-facing docs
├── validate-skills.sh                # Frontmatter validation
└── LICENSE                           # MIT
```

## Skill Invocation

When a user's request matches a skill's trigger phrases (listed in each SKILL.md frontmatter `description` field), load and follow that skill's SKILL.md instructions.

- **Match broadly.** If a user says "help me write an outreach email," that's `cold-email`. If they say "prep me for a call with a prospect," that's `discovery-call`. You don't need an exact trigger phrase match -- use judgment.
- **Load the full SKILL.md.** Don't summarize or paraphrase the skill. Read and follow its instructions, frameworks, and output formats as written.
- **Check for sales context first.** Every non-foundation skill expects `.agents/sales-context.md` to exist. If it does, load it. If it doesn't, see the error handling section below.

## Multi-Skill Workflows

Skills cross-reference each other. When a skill references another skill in its "Related Skills" section or body:

- **Suggest, don't auto-chain.** After completing a skill, tell the user which related skill to run next and why. Example: "Your buyer personas are done. Run `competitive-intel` next to build battle cards against the competitors you mentioned."
- **Don't auto-run the next skill** unless the user explicitly asks for a multi-skill workflow (e.g., "set up my full sales foundation" or "build my complete outbound sequence").
- **Respect the dependency graph.** Foundation skills feed everything. Prospecting skills feed deal execution. Deal intelligence feeds back into foundation. Follow the architecture:

```
sales-context → all skills
buyer-persona → discovery-call, demo-script, proposal-pricing
competitive-intel → objection-handling, demo-script, negotiation
lead-research → cold-email, cold-call, linkedin-outreach, direct-mail
outbound-sequence → orchestrates all channel skills
call-debrief → pipeline-review, win-loss-analysis
win-loss-analysis → competitive-intel, buyer-persona (feedback loop)
```

## Error Handling

### Missing Sales Context

If a user tries to run any non-foundation skill (anything except `sales-context`, `buyer-persona`, or `competitive-intel`) and `.agents/sales-context.md` does not exist:

1. Tell the user: "This skill works best with your sales context set up first. Run `sales-context` to define your ICP, value prop, and sales motion -- it takes about 5 minutes and every other skill reads it automatically."
2. Offer a choice: run `sales-context` now, or proceed without it (the skill will ask basic context questions inline, but the output won't be as tailored).
3. Do not silently skip the context check or pretend it doesn't matter.

### Vague or Incomplete Input

- If the user's input is too vague to produce good output, ask clarifying questions before generating. Don't guess.
- Each SKILL.md has a "Before Starting" or "Context Questions" section listing what to ask. Follow it.
- Two rounds of clarification max. If you still don't have enough, generate with what you have and flag the gaps.

## Output Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheCraigHewitt/sales-skills](https://github.com/TheCraigHewitt/sales-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
