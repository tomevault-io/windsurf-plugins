---
trigger: always_on
description: Multi-agent debate council — spawns 3 specialized sub-agents in parallel (Scholar, Engineer, Muse) for Round 1, then optional Round 2 cross-examination to challenge assumptions and strengthen the final synthesis. Configurable models and templates per role.
---


# Roundtable 🏛️ — Multi-Agent Debate Council

[![Version](https://img.shields.io/badge/version-0.4.0--beta-green)](./package.json)
[![ClawHub](https://img.shields.io/badge/ClawHub-roundtable-blue)](https://www.clawhub.ai/skills/roundtable)

Spawn 3 specialized sub-agents in parallel to tackle complex problems. You (the main agent) act as **Captain/Coordinator** — decompose the task, dispatch to specialists, run optional cross-examination, and synthesize the final answer.

## When to Use

Activate when the user says any of:
- `/roundtable <question>` or `/council <question>`
- `/roundtable setup` (interactive setup wizard)
- `/roundtable config` (show saved config)
- `/roundtable help` (command quick reference)
- "ask the council", "multi-agent", "get multiple perspectives"
- Or when facing complex, multi-faceted problems that benefit from diverse expertise

**DO NOT use for:** Simple questions, quick lookups, casual chat.

## Architecture

```
User Query
    │
    ▼
┌─────────────────────────────────┐
│  CAPTAIN (Main Agent Session)   │
│  Parse flags + assign roles     │
└────┬──────────┬─────────────────┘
     │          │          │
     ▼          ▼          ▼
┌─────────┐┌─────────┐┌─────────┐
│ SCHOLAR ││ENGINEER ││  MUSE   │
│ Round 1 ││ Round 1 ││ Round 1 │
└────┬────┘└────┬────┘└────┬────┘
     │          │          │
     └──────┬───┴───┬──────┘
            ▼       ▼
     Captain summary of all findings
            │
            ▼
┌─────────┐┌─────────┐┌─────────┐
│ SCHOLAR ││ENGINEER ││  MUSE   │
│ Round 2 ││ Round 2 ││ Round 2 │
│ critique││ critique││ critique│
└────┬────┘└────┬────┘└────┬────┘
     │          │          │
     └──────┬───┴───┬──────┘
            ▼
┌─────────────────────────────────┐
│  CAPTAIN final synthesis        │
│  consensus + dissent + confidence│
└─────────────────────────────────┘
```

## Interactive Setup

When the user sends `/roundtable setup`, run a guided, conversational setup and ask **ONE question at a time**.
Use Telegram-friendly option formatting with inline button style labels (`A)`, `B)`, `C)`).
Do not ask all steps at once.

### Step 1: Models
Ask exactly:

"🏛️ Let's set up your Roundtable! First, how do you want to configure models?
A) 🎯 Single model for all agents (simple, cost-effective)
B) 🔀 Different models per role (maximum diversity)
C) 📦 Use a preset (cheap/balanced/premium/diverse)"

Branching:
- If user picks **A** → ask: which model to use for all roles.
- If user picks **B** → ask one-by-one for: Scholar model, Engineer model, Muse model.
- If user picks **C** → ask which preset: `cheap`, `balanced`, `premium`, or `diverse`.

### Step 2: Round 2
Ask exactly:

"Do you want Round 2 cross-examination by default? (Agents challenge each other's findings — better quality but 2x cost)
A) ✅ Yes, always (recommended for important decisions)
B) ⚡ No, quick mode by default (faster, cheaper)
C) 🤷 Ask me each time"

Interpretation:
- **A** → `round2: true`
- **B** → `round2: false`
- **C** → `round2: "ask"`

### Step 3: Language
Ask exactly:

"What language should the council respond in?
A) 🇬🇧 English
B) 🇩🇪 Deutsch
C) 🇪🇸 Español
D) Other (specify)"

Interpretation:
- **A** → `language: "en"`
- **B** → `language: "de"`
- **C** → `language: "es"`
- **D** → store user-provided language value.

### Step 4: Session Logging
Ask exactly:

"Should I save council sessions for future reference?
A) ✅ Yes, save to memory/roundtable/
B) ❌ No logging"

Interpretation:
- **A** → `log_sessions: true`, `log_path: "memory/roundtable"` (fixed path, not configurable for security)
- **B** → `log_sessions: false`

**⚠️ SECURITY:** The log path is ALWAYS `memory/roundtable/` relative to the workspace. Custom paths are NOT allowed to prevent path traversal attacks.

### Step 5: Confirmation + Write
Show a concise summary of all collected choices and ask user to confirm.
Only after confirmation, write `config.json` in this skill directory.

Required command behavior:
- `/roundtable config` → Show current `config.json` if it exists, otherwise: `No config found, run /roundtable setup to configure.`
- `/roundtable help` → Show quick reference:
  - `/roundtable <question>` — ask the council
  - `/roundtable setup` — interactive setup wizard
  - `/roundtable config` — show current config
  - `/roundtable help` — this help

## Model Configuration

Users can specify models per role. Parse from the command or use defaults.

### Modes

**Single-model mode** (same model, different perspectives):
```
/roundtable <question>
/roundtable <question> --all=sonnet
```
All 3 agents use the SAME model but with different system prompts and focus areas. This is the simplest setup — the value comes from the **different perspectives**, not necessarily different models.

**Multi-model mode** (different models per role):
```
/roundtable <question> --scholar=codex --engineer=codex --muse=sonnet
```
Each agent runs on a different model optimized for its role. This is the power configuration — different models bring genuinely different reasoning patterns.

### Syntax
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robbyczgw-cla/roundtable](https://github.com/robbyczgw-cla/roundtable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
