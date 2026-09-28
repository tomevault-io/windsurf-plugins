---
trigger: always_on
description: CLI framework for managing and distributing 50+ AI agent skills across 11 AI agents (3 dedicated + 8 universal).
---


# AI Agents Skills Framework

CLI for creating, managing, and distributing AI agent skills across 11 AI agents. Local-first architecture with symlink-based installation, dependency resolution, and token-efficient model instructions.

## How to Use Skills (MANDATORY WORKFLOW)

This project has skills installed in your model's skills directory. Follow this protocol for ALL coding tasks:

### Step 1: Find the Trigger

Check the "Mandatory Skills" table below. Match your task to the "Trigger" column.

### Step 2: Read the Skill

Find your agent below and use the corresponding path:

| Agent | Skills path |
|-------|------------|
| Claude Code | `.claude/skills/{skill-name}/SKILL.md` |
| Antigravity | `.agent/skills/{skill-name}/SKILL.md` |
| OpenClaw | `skills/{skill-name}/SKILL.md` |
| Amp, Cline, Codex, Cursor, Gemini CLI, GitHub Copilot, Kimi, OpenCode | `.agents/skills/{skill-name}/SKILL.md` |

**Shortcut:** All skill source files live at `skills/{skill-name}/SKILL.md` — if your agent can't resolve symlinks, read from there directly.

### Step 3: Read Dependencies

Every skill lists dependencies in its frontmatter (`metadata.skills`). Read each direct dependency before proceeding.

**Example:** `react` skill depends on: `a11y`, `typescript`, `javascript`, `architecture-patterns`

Read these 4 direct dependencies. Dependencies are resolved transitively - when you read `typescript`, you'll see it depends on `javascript`, which depends on `code-conventions`. The dependency chain ensures you have all required context.

### Step 4: Apply Patterns

- Follow "Critical Patterns" marked with ✅ REQUIRED
- Use "Decision Tree" for implementation choices
- Reference inline code examples

### Example Workflow

**Task:** "Create TypeScript interface for User model"

1. **Check table below** → Trigger: "TypeScript types/interfaces" → Skill: `typescript`
2. **Read:** `skills/typescript/SKILL.md` (or your agent's path from Step 2)
3. **Check frontmatter** → Dependencies: `javascript`
4. **Read dependency:**
   - `skills/javascript/SKILL.md` (which depends on `code-conventions`)
5. **Apply patterns:** Use `interface` (not `type`), PascalCase names, export from `types/` directory

## Mandatory Skills

**Path:** Use the table in Step 2 above to find the correct path for your agent.

| Trigger                          | Skill            |
| -------------------------------- | ---------------- |
| Create or modify skills          | skill-creation   |
| Create agent definitions         | agent-creation   |
| Code review or improvements      | critical-partner |
| Coding standards                 | code-conventions |
| TypeScript code                  | typescript       |
| Node.js / CLI development        | nodejs           |
| Writing unit tests               | unit-testing     |
| Jest test suite or config        | jest             |
| Exploring ideas or approaches    | brainstorming    |
| Astro pages, layouts, components | astro                  |
| Tailwind utilities or styling    | tailwindcss            |
| Accessibility or UI components   | a11y                   |
| Frontend workflow or components  | frontend-dev           |
| UI/UX decisions or design review | interface-design       |
| Writing architecture or spike docs | tech-docs            |
| Commit messages or documentation | technical-communication|
| Creating reference files         | reference-creation     |
| Syncing skills across models     | skill-sync             |
| Debugging errors or root cause   | systematic-debugging   |
| HTML markup or structure         | html                   |
| CSS properties or animations     | css                    |
| JavaScript patterns or scripts   | javascript             |
| Writing skill content in English | english-writing        |
| Formal code review checklist     | code-review            |
| Processing incoming review feedback | receiving-code-review |
| Planning implementation tasks    | writing-plans          |
| Verifying task completion        | verification-protocol  |
| Creating model prompt files      | prompt-creation        |
| Agent overthinking or hesitation | sharp-execution        |
| Minimize response tokens         | lean-output            |
| Shipping or closing a branch     | ship-branch            |
| Clarify requirements before acting | grill-me             |
| Summarize session for new chat   | context-handoff        |
| Auth, JWT, OAuth, password hashing | authentication       |
| Dockerfiles or containerization  | docker                 |
| GraphQL schemas or resolvers     | graphql                |
| Documenting a design system      | design-system-spec     |
| React code quality review        | react-best-practices   |
| Astro site quality review        | astro-best-practices   |
| CSS architecture review          | css-best-practices     |
| Node.js service quality review   | nodejs-best-practices  |

## Skills Reference

50+ skills organized by category (exact count shown as nearest lower multiple of 50: 50+, 100+, 150+, etc.):

- **Frameworks:** React, Next.js, Astro, Express, Nest, Hono, React Native, Expo
- **Best Practices:** React Best Practices, Astro Best Practices, CSS Best Practices, Node.js Best Practices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joabgonzalez/ai-agents-skills](https://github.com/joabgonzalez/ai-agents-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
