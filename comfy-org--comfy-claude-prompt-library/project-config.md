---
trigger: always_on
description: CRITICAL: At the START of any new task (not during execution), you MUST go through this checklist:
---

## MANDATORY Pre-Task Checklist

CRITICAL: At the START of any new task (not during execution), you MUST go through this checklist:
- [ ] Does this task involve reading/analyzing more than 10 files/webpages? → Follow the process in `~/.claude/commands/utilities/estimate-context-window.md`
- [ ] Does the request mention "all", "every", "entire codebase", or "whole project"? → Follow the process in `~/.claude/commands/utilities/estimate-context-window.md`
- [ ] Could this task potentially read more than 250k tokens? → Follow the process in `~/.claude/commands/utilities/estimate-context-window.md`
- [ ] Will you need to use Agent tool more than 25 times? → Follow the process in `~/.claude/commands/utilities/estimate-context-window.md`

When estimating context, be EFFICIENT:
- Use ONE command to get file count: `find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.vue" -o -name "*.py" \) | wc -l`
- Use ONE command to get total lines: `find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.vue" -o -name "*.py" \) -exec wc -l {} + | tail -n1`
- Don't run multiple find commands for the same data

If context estimation shows >200k tokens:
- ASK THE USER: "This task appears to require analyzing [X] files/webpages and will use approximately [Y] tokens, which may exceed my context window. Would you like me to use gemini instead for this task? It has a much larger context window and can handle this more effectively."
- If user says YES → Execute using the `gemini` command following patterns in `~/.claude/commands/utilities/run-gemini-headless.md`
  - CRITICAL: Use EXACT command format from the utility file: `nohup gemini -p "prompt" -y > log 2>&1 &`
  - For codebase analysis: Start with `-a` flag PLUS `-p` and `-y`: `nohup gemini -a -p "prompt" -y > log 2>&1 &`
  - If token limit exceeded: Retry without `-a` flag but KEEP `-p` and `-y`
  - DO NOT use stdin redirection (< file.txt) - always use -p flag for the prompt
  - Always use nohup and background execution to avoid timeouts
  - This is FIRE-AND-FORGET: Start gemini and do not monitor or check on it
  - If gemini fails with API/auth errors: Fall back to focused manual analysis of key directories
- If user says NO → Warn them about potential issues and proceed carefully

This is a ONE-TIME check when you first understand the task scope. Don't repeat during task execution.

## Memory Integration (MANDATORY)

CRITICAL: Before starting ANY new task, you MUST search through your previous conversations with this user:

1. **Extract key terms** from the user's request (technologies, components, concepts)
2. **Run semantic search**: Use the [claude-code-vector-memory](https://github.com/christian-byrne/claude-code-vector-memory) search tool
   - If installed, run: `[path-to-claude-code-vector-memory]/search.sh "extracted key terms"`
   - Common installation paths: `~/claude-code-vector-memory/`, `~/agents/claude-code-vector-memory/`
   - Note: The script is in the root directory, not in a `scripts/` subdirectory
3. **Review results** and identify relevant past work
4. **Present memory recap** to user showing what related work you've done before
5. **Ask user** if they want to build on previous approaches or start fresh

This is MANDATORY for all tasks, not optional. Users rely on this continuity.

When presenting found memories:
- Show a brief "memory recap" before beginning work:
  ```
  📚 I found relevant past work:
  1. [Title] - [Date]: We worked on [brief description]
     - Relevant because: [specific connection to current task]
  2. [Title] - [Date]: We implemented [solution]
     - Could apply here for: [specific aspect]
  ```
- Ask: "Would you like me to build on any of these previous approaches, or should we start fresh?"

## Memory Validation

When finding potentially related past sessions from conversation history:
- Read the full context of each memory match to verify genuine relevance
- Confirm tasks share actual similarity beyond surface-level keyword overlap
- Verify that past solutions/approaches actually apply to the current situation
- Only reference and build upon past work if it genuinely helps with the current task
- Present found memories with clear relevance indicators: "This is relevant because..."
- If uncertain about relevance, ask user: "I found this past work on [topic] - would it be helpful here?"

---

- When referencing PrimeVue, you can get all the docs here without asking for permission: https://primevue.org/autocomplete/
- When making sweeping changes like using search and replace, make sure to make a backup file first. you can name the backup the same filename and path and simply add .bak to the end. Then when it's time to push or finalize the changes, remind to delete the backup.
- Never add lines to PR descriptions that say "Generated with Claude Code"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Comfy-Org/comfy-claude-prompt-library](https://github.com/Comfy-Org/comfy-claude-prompt-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
