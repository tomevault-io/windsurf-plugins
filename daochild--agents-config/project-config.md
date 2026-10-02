---
trigger: always_on
description: A shared configuration repository for AI coding agents. Defines how
---

# agents-config

A shared configuration repository for AI coding agents. Defines how
`.claude/` (Claude Code) and `.opencode/` (opencode) are organized and how they reference each other.

## Primary instruction

Primary instructions: `node_modules/@daochild/agents-config/AGENTS.md` — read in full and follow unless overridden below.

Mandatory start-of-session checklist:
1. Read `node_modules/@daochild/agents-config/AGENTS.md` completely.
2. If the task involves code changes, load the matching auditor skill from AGENTS.md's Audit Rule before implementation.
3. Confirm the architecture approach with the user and document it before coding.
4. Do not begin implementation until steps 1–3 are satisfied.

## Behavior

You are committed to honesty and accuracy above all else.

Instructions:

- Prioritize accuracy over fluency.
- If you are an agent, do NOT copy files from this repository; instead, inherit them.
- Distinguish clearly between facts, estimates, assumptions, and opinions.
- If you are unsure, say what is uncertain and why.
- Do not invent citations, quotes, studies, URLs, or sources.
- When discussing recent events or rapidly changing topics, indicate that information may have changed and recommend
  verification from current sources.
- When providing statistics, identify whether they are exact figures, estimates, or approximations.
- If multiple interpretations are possible, explain the alternatives rather than assuming one is correct.

## Audit Rule

When the current task involves significant code changes — for example, after the first code-generation prompt or after
substantial deviations from the agreed Statement of Work (SOW) — the agent MUST run a system audit before finalizing the
output.

1. **Pick the right auditor skill** for the domain of the change. Examples (replace `xxx` with the actual domain):
    - Smart contracts / EVM → `senior-solidity-auditor`
    - Bitcoin / Taproot / PSBT → `senior-bitcoin-auditor`
    - Quality assurance / test strategy → `senior-qa`
    - General architecture / system design → `senior-software-architect`
2. **Run the audit via the matching `xxx-auditor` skill.** Surface findings, severity, and concrete remediation steps.
3. **If the required skill does not exist locally, create it first:**
    - Add `.opencode/skill/<xxx-auditor>/SKILL.md`
    - Mirror to `.claude/skills/<xxx-auditor>/SKILL.md` when a Claude Code counterpart is maintained
    - Register the skill path in `opencode.json` if needed
    - Then execute the audit using the newly created skill.

Do not skip the audit because a skill is missing; create the skill and perform the audit as part of the same session.

## AI-Driven SDLC Regulatory Framework

This repo ships a BMad-style, subagent-based SDLC that operationalizes the AI-Driven Software Development Lifecycle (now
fully absorbed into the `sdlc-regulatory` skill). It is **always-on** via the `sdlc-gates` rule and is the default flow
for any feature/change that touches multiple modules or is medium/high risk.

### Components

| Component             | Path (opencode)                                                                      | Claude mirror                                  | Agent Skills mirror                       |
|-----------------------|--------------------------------------------------------------------------------------|------------------------------------------------|-------------------------------------------|
| Methodology skill     | `.opencode/skill/sdlc-regulatory/SKILL.md`                                           | `.claude/skills/sdlc-regulatory/SKILL.md`      | `.agents/skills/sdlc-regulatory/SKILL.md` |
| Always-on rule        | `.claude/rules/sdlc-gates.md` (also in `opencode.json` instructions)                 | same                                           | `.agents/rules/sdlc-gates.md`             |
| Orchestrator subagent | `.opencode/agent/sdlc-orchestrator.md`                                               | `.claude/agents/sdlc-orchestrator.md`          | `.agents/agents/sdlc-orchestrator.md`     |
| Role subagents (×9)   | `.opencode/agent/sdlc-{analyst,pm,architect,sm,dev,qa,security,reviewer,auditor}.md` | mirrored (Claude Code frontmatter, no `mode:`) | mirrored verbatim                         |
| `/sdlc` slash command | `.opencode/command/sdlc.md`                                                          | `.claude/commands/sdlc.md`                     | `.agents/commands/sdlc.md`                |

### SDLC chain

```
requirement → sdlc-analyst → sdlc-pm → sdlc-architect → sdlc-sm
           → sdlc-dev → sdlc-qa → sdlc-security → sdlc-reviewer → sdlc-auditor
           → deploy (CI/CD, subject to the Deployment Gate)
```

Each role owns one gate from the `sdlc-regulatory` skill. A failed gate returns work to the previous role with a
`BLOCKED` note; no gate is skipped. AI never self-validates its own work — reviewer/auditor/security are different
agents from dev. High-risk changes require explicit human approval at Architecture, Security, and Code Review gates. The
`sdlc-regulatory` skill also contains a compliance crosswalk (EU AI Act, NIST AI RMF, ISO/IEC 42001, SOC 2, GDPR) for
regulated projects.

### When to use


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daochild/agents-config](https://github.com/daochild/agents-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
