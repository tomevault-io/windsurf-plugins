---
trigger: always_on
description: - This repo defines GitHub Copilot agent behavior and AB-100 skills. There is no app code; the primary artifacts are agent definitions and skill specs.
---

# Copilot instructions

## Repository purpose

- This repo defines GitHub Copilot agent behavior and AB-100 skills. There is no app code; the primary artifacts are agent definitions and skill specs.
- This repo also serves as the companion material for Tim Warner's O'Reilly Live Learning course **Agentic AI Business Solutions Architect**. The cert-buddy agent and the course scaffolding share this workspace.

## Key locations

- Agent definition: `.github/agents/ab100-cert-buddy-agent.agent.md`
- Skills: `.github/skills/*/SKILL.md`
- Prompts: `.github/prompts/ab100-quiz.prompt.md`, `.github/prompts/ab100-lab.prompt.md`, `.github/prompts/ab100-plan.prompt.md`
- MCP servers (workspace only): `.vscode/mcp.json`
- AB-100 objectives (canonical, verbatim Microsoft Learn sync): `docs/ab100-exam-objectives.md`
- Style and naming rules: embedded directly in the agent and skill files (Microsoft Worldwide Learning Exam Writing Style Guide for items, Microsoft Writing Style Guide for prose).

## Skill and agent conventions

- Each skill file is a single YAML frontmatter block followed by Markdown.
- The skill name used by agents must match the YAML frontmatter name in the skill file, not the folder name.
- Skills are auto-discovered from `.github/skills/` folders. The agent references them by name in its Markdown body; there is no `skills:` field in agent YAML frontmatter.
- Agent files use YAML frontmatter with tools lists. Keep tool IDs scoped to MCP servers defined in `.vscode/mcp.json`.
- Prompt files (in `.github/prompts/`) reference agents by the `name` field in the agent's YAML frontmatter via the `agent:` key. The current value is `ab100-cert-buddy-agent`. If the agent is renamed, update all prompt files that reference it.
- Current skills:
  - `ab100-item-creator` (exam item generation)
  - `ab100-lab-creator` (design/ALM/governance lab generation)
  - `ab100-study-planner` (personalized study plan generation)

## MCP server IDs

- `ab100buddy-mslearn` -- Microsoft Learn MCP (free, no API key; provides `microsoft_docs_search`, `microsoft_docs_fetch`, `microsoft_code_sample_search`)

### How users set up the MCP server

The Microsoft Learn MCP server requires no API keys, logins, or sign-ups. It is configured in `.vscode/mcp.json` as an HTTP server pointing to `https://learn.microsoft.com/api/mcp`. Users can also search for "@mcp learn" in the VS Code Extensions marketplace for one-click install.

## Grounding and validation rules

- Questions must be grounded in Microsoft Learn content first. Use the Microsoft Learn MCP server to retrieve current Learn documentation and code samples.
- Design, ALM, and governance labs must be grounded in Microsoft Learn for correct configuration and any CLI, PowerShell, YAML, or Bicep syntax.
- If a request mixes questions and labs, split the output and apply the correct skill to each section.
- Grounding chain: Microsoft Learn MCP (`microsoft_docs_search` for discovery, `microsoft_docs_fetch` for full detail, `microsoft_code_sample_search` for code accuracy).

## Answer choice randomization (non-negotiable)

When generating practice questions, the correct answer MUST be randomized across A, B, C, and D. Never default to any single letter position.

## Fictional company randomization (non-negotiable)

Use only WWL-approved fictional company names (the full list with approved URLs is embedded in `.github/agents/ab100-cert-buddy-agent.agent.md` and in the `ab100-item-creator` and `ab100-lab-creator` skills). Always use the entire company name on every mention (write *Litware, Inc.*, not *Litware*). Randomize across the full list. Do not default to Contoso, Ltd. for every scenario.

## Terminology (non-negotiable)

Always use current Microsoft product names. Never use a retired or legacy name, even if the user does. Silently map to the current name.

| Retired / legacy name | Current name |
| --- | --- |
| Azure Active Directory (Azure AD) | Microsoft Entra ID |
| Azure AD tenant | Microsoft Entra tenant |
| Azure AD Conditional Access | Microsoft Entra Conditional Access |
| Azure AD B2B / B2C | Microsoft Entra External ID |
| Azure AD PIM | Microsoft Entra Privileged Identity Management |
| Azure OpenAI Service (standalone, post-rebrand) | Azure OpenAI in Microsoft Foundry |
| Azure AI Studio | Microsoft Foundry |
| Azure AI Foundry | Microsoft Foundry (official rebrand) |
| Azure AI Foundry Tools | Microsoft Foundry Tools |
| Copilot Studio classic topics | Copilot Studio topics |
| Power Virtual Agents | Microsoft Copilot Studio |
| Copilot for Microsoft 365 | Microsoft 365 Copilot |
| Copilot for Sales / Copilot for Service | Microsoft 365 Copilot for Sales / Microsoft 365 Copilot for Service |
| Dataverse for Teams | Microsoft Dataverse |

If Microsoft Learn shows a different current name than what appears above, prefer the Learn name. Always check the [AB-100 study guide](https://learn.microsoft.com/credentials/certifications/resources/study-guides/ab-100) and active product pages first.

## Interactive question delivery

When the user asks for a practice question (one or more items):

1. Present **only** the metadata, scenario stem, and answer choices.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timothywarner-org/ab100](https://github.com/timothywarner-org/ab100) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
