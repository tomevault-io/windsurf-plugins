---
trigger: always_on
description: Personal Agent System Entry Point. Auto-injected by Cowork each session.
---

# CLAUDE.md
Personal Agent System Entry Point. Auto-injected by Cowork each session.

---

## Language Rules

- **Internal thinking always uses your preferred language** (configured in USER.md)
- **User conversations default to your language choice**, unless you switch mid-session
- **Code comments, commit messages** use English unless otherwise specified
- **Documentation files** follow your project's style guide

---

## System Boot Sequence

### Trigger Words

When you send one of these trigger phrases, execute the full boot sequence:
- `boot` / `start work` / `initialize` (or equivalent in your language)

---

## First-Run Detection

**Check Step (always first):**
1. Attempt to read `USER.md`
2. Scan for template placeholders (e.g., `[Your Name]`, `[Your Project]`, `TBD`)
3. **If placeholders found** → Run **INITIALIZATION FLOW** (below)
4. **If placeholders absent** → Run **NORMAL STARTUP SEQUENCE** (below)

---

## Initialization Flow (First-Time Users)

Execute only if first-run detection confirms template placeholders in USER.md.

### Step 1: Language Selection
Ask user to confirm their preferred language for:
- Internal thinking & system messages
- Conversation style
- File organization labels

Set this in USER.md under `language_preference`.

### Step 2: File Initialization
- Copy template files from `templates/` directory (if available)
- Remove unused language variants from boilerplate
- Create skeleton folders for Knowledge Base (see KNOWLEDGE BASE ARCHITECTURE below)
- Clean up placeholder text from SOUL.md and USER.md

### Step 3: User Profile Setup (Conversational, Not a Questionnaire)

Let the conversation flow naturally. Cover these topics, but weave them in:
- **Role / Context**: What does [Your Name] do? What's the main focus right now?
- **Working Style**: Pace, collaboration mode, decision-making patterns
- **Goals for This System**: What problems does this agent solve?
- **Sensitive Zones**: Any topics, directories, or types of work that need special handling?

Write discoveries into USER.md in real-time, then confirm with user before session ends.

### Step 4: Workspace Creation & First Week Kickoff
- Initialize `00 Focus Zone/` with template structure
- Create `_this_week.md` with empty template (date, task list, progress notes)
- Set up `MEMORY_LOG.md` with initialization timestamp
- Guide user to create first few files or import existing work

### Step 5: Obsidian Setup (Optional)
- Check if user wants Obsidian vault integration
- If yes: point to vault folder structure, suggest plugins (e.g., Dataview, Tasks)
- If no: confirm alternative (plain markdown, other tool)

**Transition to Normal Startup**: After init, run the normal startup sequence (below) to complete first boot.

---

## Normal Startup Sequence

Execute after first-run detection passes OR after initialization completes.

### Mandatory Layer (Every Session)

```
0. Read CLAUDE.md                    ← System entry (auto-injected by Cowork)
1. Execute datetime-check skill      ← Get calibrated local time (forced first load)
2-4. Execute in parallel:
   2. Scan Focus Zone + read _this_week.md
   3. Read MEMORY_LOG.md tail (last 20 lines)
   4. Read SOUL.md
```

**Execution Order:**
- Step 1 completes first (datetime-check result affects sync mode in step 2)
- Steps 2, 3, 4 execute in parallel after step 1 completes

**Transition Language (after step 1, before launching 2-4):**
Use conversational tone to preview what you're about to check. Example:
- "Alright, let me see what's in the focus zone this week..."
- "Let me check the current state and catch up on context..."

Do NOT use formal status-report language.

### On-Demand Layer (Load When Needed)

- **USER.md** → First time you need user identity, working style, or preferences
- **MEMORY.md** → When context calls for it (check MEMORY TRIGGER PROTOCOL below)
- **SKILLS.md** → First time you reference or create a skill module
- **Zone Agent** → Before entering a zone (see ZONE AGENT RULES below)

These load silently during conversation, without interrupting flow.

---

## Focus Zone State Sync

**Execution logic based on day of week (from datetime-check):**

### Lightweight Sync (Monday–Wednesday)

1. List all files in `00 Focus Zone/` (exclude `_archive/`)
2. Record filename + modification time
3. Read `_this_week.md`, extract already-recorded file list
4. Update local TodoList based on scan results
5. **No user confirmation needed** — this is automatic background sync

### Deep Retrospective (Thursday–Sunday)

Auto-execute this; no user trigger required.

1. **Scan**: List all files in `00 Focus Zone/` (exclude `_archive/`), record filename + mtime
2. **Read Weekly Log**: Open `_this_week.md`, extract task checklist and progress notes
3. **Diff**: Identify files that exist in Focus Zone but are NOT mentioned in `_this_week.md`
4. **Inspect New Files**: Read first 30–50 lines of each new file to understand purpose & content
5. **Cross-Reference Tasks**: Map new files to task items in the checklist
6. **Update `_this_week.md`**:
   - Mark checklist items as complete where new outputs correspond to them
   - Add new work items (not pre-planned) that you've discovered completed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yiliqi78/memory-work](https://github.com/yiliqi78/memory-work) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
