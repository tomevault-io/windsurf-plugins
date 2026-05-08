---
trigger: always_on
description: > This file defines how Claude Code behaves in this workspace.
---

# Bilge Development Kit - Claude Code Rules

> This file defines how Claude Code behaves in this workspace.
> Adapted from GEMINI.md for Claude Code's native tooling.

---

## MEMORY BANK (Session Start)

**At the start of every session**, check if `.agent/.claude/memory/` directory exists and read any available memory files:
- `MEMORY-activeContext.md` — current work state, recent sessions
- `MEMORY-patterns.md` — established code patterns and conventions
- `MEMORY-decisions.md` — architecture decision records
- `MEMORY-troubleshooting.md` — solved issues and lessons learned

Use this context to avoid re-discovering information. If memory files don't exist, suggest running `/remember` to initialize the Memory Bank.

---

## CRITICAL: AGENT & SKILL PROTOCOL (START HERE)

**MANDATORY:** You MUST read the appropriate agent file and its skills BEFORE performing any implementation.

### 1. Modular Skill Loading Protocol

```
Agent activated -> Check frontmatter "skills:" -> Read SKILL.md (INDEX) -> Read specific sections
```

- **Selective Reading:** DO NOT read ALL files in a skill folder. Read `SKILL.md` first, then only read sections matching the user's request.
- **Rule Priority:** P0 (CLAUDE.md root) > P1 (Agent .md) > P2 (SKILL.md). All rules are binding.

### 2. Enforcement Protocol

1. **When agent is activated:**
   - Read Rules -> Check Frontmatter -> Load SKILL.md -> Apply All.
2. **Forbidden:** Never skip reading agent rules or skill instructions. "Read -> Understand -> Apply" is mandatory.

---

## REQUEST CLASSIFIER (STEP 1)

**Before ANY action, classify the request:**

| Request Type     | Trigger Keywords                           | Active Tiers                         | Result                      |
| ---------------- | ------------------------------------------ | ------------------------------------ | --------------------------- |
| **QUESTION**     | "what is", "how does", "explain"           | TIER 0 only                          | Text Response               |
| **SURVEY/INTEL** | "analyze", "list files", "overview"        | TIER 0 + Explore subagent            | Session Intel (No File)     |
| **SIMPLE CODE**  | "fix", "add", "change" (single file)       | TIER 0 + TIER 1 (lite)              | Inline Edit                 |
| **COMPLEX CODE** | "build", "create", "implement", "refactor" | TIER 0 + TIER 1 (full) + Agent tool | **{task-slug}.md Required** |
| **DESIGN/UI**    | "design", "UI", "page", "dashboard"        | TIER 0 + TIER 1 + Agent tool        | **{task-slug}.md Required** |

---

## INTELLIGENT AGENT ROUTING (STEP 2 - AUTO)

**ALWAYS ACTIVE: Before responding to ANY request, automatically analyze and select the best agent(s).**

### Auto-Selection Protocol

1. **Analyze (Silent)**: Detect domains (Frontend, Backend, Security, etc.) from user request.
2. **Select Agent(s)**: Choose the most appropriate specialist(s).
3. **Read Agent File**: Use Read tool on `.agent/agents/{agent-name}.md`.
4. **Load Skills**: Read SKILL.md files listed in the agent's frontmatter `skills:` field.
5. **Apply**: Generate response using the selected agent's persona and rules.

### AGENT ROUTING CHECKLIST (MANDATORY BEFORE EVERY CODE/DESIGN RESPONSE)

| Step | Check | If Unchecked |
|------|-------|--------------|
| 1 | Did I identify the correct agent for this domain? | STOP. Analyze request domain first. |
| 2 | Did I READ the agent's `.md` file? | STOP. Read `.agent/agents/{agent}.md` |
| 3 | Did I load required skills from agent's frontmatter? | STOP. Check `skills:` field and read them. |
| 4 | Am I applying the principles, not just copying patterns? | STOP. Understand WHY, then apply. |

**Failure Conditions:**
- Writing code without identifying an agent = PROTOCOL VIOLATION
- Ignoring agent-specific rules = QUALITY FAILURE

---

## TIER 0: UNIVERSAL RULES (Always Active)

### Language Handling

When user's prompt is NOT in English:
1. **Internally translate** for better comprehension
2. **Respond in user's language** - match their communication
3. **Pay extra attention to native characters** — e.g. Turkish: always use ç, ğ, ı, İ, ö, ş, ü correctly. Never fall back to ASCII equivalents.
4. **Code comments/variables** remain in English

### Clean Code (Global Mandatory)

**ALL code MUST follow `skills/clean-code` rules. No exceptions.**

- **Code**: Concise, direct, no over-engineering. Self-documenting.
- **Testing**: Mandatory. Pyramid (Unit > Int > E2E) + AAA Pattern.
- **Performance**: Measure first. Adhere to 2025 standards (Core Web Vitals).
- **Infra/Safety**: 5-Phase Deployment. Verify secrets security.

### File Dependency Awareness

**Before modifying ANY file:**
1. Check `CODEBASE.md` if it exists for file dependencies
2. Identify dependent files
3. Update ALL affected files together

### Hooks (Automated Guardrails)

**Active automatically via `.claude/settings.json`.** No manual intervention needed.

- **PreToolUse:Bash** → `dangerous_cmd_check.sh` blocks destructive commands
- **PreToolUse:Write|Edit** → `secret_scanner.sh` blocks hardcoded secrets
- **PostToolUse:Edit|Write** → `lint_check.sh` runs lint after file changes
- **Stop** → `session_save.sh` persists session context

### Always-On Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bugrabilge/bilge-development-kit](https://github.com/bugrabilge/bilge-development-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
