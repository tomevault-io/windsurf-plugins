---
trigger: always_on
description: >
---


# Spec Driven Development Skill

This skill guides the user through creating a complete Spec Driven Development environment:
- `requirements.md` — what the system must do
- `design.md` — how it will be built
- `tasks.md` — atomic, ordered implementation steps
- AI agent configuration files for Claude Code, Cursor, Windsurf, GitHub Copilot, and Aider

**Reference files** (read when needed):
- `references/sdd-curriculum.md` — full teaching curriculum, newbie to hero
- `references/capabilities-and-cross-ai.md` — Claude Code capabilities, limitations, cross-AI strategy
- `references/templates.md` — all spec file and AI config templates

---

## When to Read Reference Files

| Situation | Read |
|---|---|
| User is new to SDD and needs explanation | `sdd-curriculum.md` Levels 1–3 |
| User asks about Claude Code + SDD specifics | `capabilities-and-cross-ai.md` Part 1 |
| User wants cross-AI config files | `capabilities-and-cross-ai.md` Part 2–3 |
| You need template content to generate files | `templates.md` |

---

## Workflow: First Run (New Project)

### Step 1: Interview the User

**Conversational rule (non-negotiable):** Ask exactly ONE question, wait for the answer, then ask the next. Never present questions as a numbered list or bullet list — that feels like a form, not a conversation. A newbie who sees three bullets may answer only the first or feel overwhelmed.

**The four required answers before any file is generated:**
1. What the project does and who uses it
2a. Tech stack (language, framework, database)
2b. Deployment target (Railway, Fly.io, AWS, Vercel, etc. — offer to suggest if unknown)
3. Which AI coding tools they use (Claude Code, Cursor, Windsurf, Copilot, Aider, other)

Stack and deployment are separate required answers because one does not imply the other. "Node.js" tells you nothing about deployment. "Railway" tells you nothing about the language. Both must be known independently before generation proceeds.

**If the user's opening message already answers some but not all of these**, acknowledge what you know and ask only for what's missing — still one question at a time.

**Optional follow-ups** (ask only when the user's answer raises real ambiguity):
- "Are there performance, security, or accessibility constraints?"
- "What is explicitly out of scope for this first version?"

Example of correct pacing:
> User: "I want to start a new project"
> Claude: "Happy to help you spec it out. What does the project do — who are its users and what's the core job it performs?"
> User: "It's a task manager for small dev teams"
> Claude: "Got it. What's your tech stack — language, framework, database?"
> User: "Node.js, Express, PostgreSQL"
> Claude: "And where are you planning to deploy? (Railway, Fly.io, AWS — or I can suggest one)"
> User: "Railway"
> Claude: "Last thing — which AI coding tools do you use? (Claude Code, Cursor, Copilot, Windsurf, something else?)"

Example of partial-answer handling:
> User: "I'm building a task management API in Node.js"
> Claude: "Good start — Node.js noted. What database are you planning to use?"
> User: "PostgreSQL"
> Claude: "And where are you deploying? (Railway, Fly.io, AWS — or I can suggest one)"
> User: "Railway"
> Claude: "Last thing — which AI coding tools do you use?"
> *(stack partially given — ask for each missing part one at a time, never bundle them)*

### Generation Gate (enforced between Step 1 and Step 2)

**HARD RULE: No file of any kind may be generated until all four interview answers are in hand.**

This gate applies to every output path — `requirements.md`, `design.md`, `tasks.md`, `.cursorrules`, `CLAUDE.md`, and all other config files. There are no exceptions, including when:
- The user says "create requirements for X" (X is a product category, not a full description)
- The user says "make a cursorrules file" (a concrete deliverable name is not permission to skip the interview)
- The user says "set up Cursor and Claude Code" (naming tools is not the same as providing project context)
- The user gives a partial stack answer like "Node.js" (stack ≠ deployment — both are required separately)
- The user provides some but not all of the four required answers (ask for the rest one at a time)

If any answer is missing, ask for it conversationally before proceeding. Do not generate placeholder files with `{{UNFILLED}}` tokens. Do not generate plausible-sounding fake requirements from a product category alone.

**Gate check before generating any file:**
```
□ Do I know what the project does and who uses it?    → if not, ask first
□ Do I know the tech stack (language/framework/db)?   → if not, ask first
□ Do I know the deployment target?                    → if not, ask first
□ Do I know which AI tools the user uses?             → if not, ask first
Only when all four are ✓ → proceed to Step 2
```

---

### Step 2: Generate `requirements.md`

Read `references/templates.md` for the template. Fill in all `{{PLACEHOLDERS}}`.

**Quality rules for requirements.md:**
- Every functional requirement uses "shall" language (REQ-xxx: Actor shall verb object)
- Every requirement has a concrete acceptance criterion
- NFRs have a measurable metric (not "fast" — use "< 200ms at p95")
- Out of scope section is non-empty (if user didn't provide it, infer likely assumptions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FredAntB/Spec-Driven-Development](https://github.com/FredAntB/Spec-Driven-Development) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
