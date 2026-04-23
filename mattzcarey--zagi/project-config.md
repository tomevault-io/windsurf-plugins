---
trigger: always_on
description: This document describes the process for adding new git commands to zagi.
---

# zagi

This document describes the process for adding new git commands to zagi.

## Requirements

- Zig 0.15+
- Bun (for integration tests)

## Style

- zagi is always lowercase
- No emojis in code or output
- Concise output optimized for agents

## Committing with Prompts

When committing code, **always include the complete user prompt** that created the change:

```bash
git commit -m "<commit-message>" --prompt "<complete-user-prompt-for-this-commit>"
```

The `--prompt` should contain the **full user request** that led to this commit, not a summary. This preserves context for code review.

Example:
```bash
git commit -m "Add logout button" --prompt "Add a logout button to the header. When clicked it should clear the session and redirect to /login"
```

When `--prompt` is used, zagi automatically stores metadata in git notes:
- `refs/notes/agent` - detected AI agent (claude, opencode, cursor, windsurf, vscode, terminal)
- `refs/notes/prompt` - the user prompt text
- `refs/notes/session` - full session transcript (for Claude Code, OpenCode)

View metadata with:
```bash
git log --prompts   # show prompts (truncated to 200 chars)
git log --agent     # show which AI agent made each commit
git log --session   # show session transcript (paginated, first 20k bytes)
git log --session --session-offset=20000  # continue from byte 20000
```

### Agent Mode

Agent mode is automatically enabled when running inside AI tools:
- Claude Code (sets `CLAUDECODE=1`)
- OpenCode (sets `OPENCODE=1`)
- VS Code, Cursor, Windsurf (detected from terminal environment)

You can also enable it manually:
```bash
export ZAGI_AGENT=my-agent
```

When agent mode is active:
1. `git commit` will fail without `--prompt`, ensuring all AI-generated commits have their prompts recorded
2. Destructive commands are blocked to prevent data loss

## Agent Subcommands

zagi provides two agent subcommands for autonomous task execution using the RALPH pattern (Recursive Agent Loop Pattern for Humans).

### zagi agent plan

Starts an interactive planning session where an AI agent collaborates with you to design and create tasks.

```bash
# Start an interactive session (agent will ask what you want to build)
zagi agent plan

# Start with initial context
zagi agent plan "Add user authentication with JWT"

# Preview the prompt without executing
zagi agent plan --dry-run
```

The planning agent follows an interactive protocol:
1. **Explore codebase**: Reads AGENTS.md and relevant code to understand architecture
2. **Ask clarifying questions**: Asks about scope, constraints, and preferences before drafting any plan
3. **Propose plan**: Presents a numbered implementation plan for your review
4. **Create tasks**: Only creates tasks after you explicitly approve the plan

This collaborative approach ensures the agent gathers all necessary context before committing to a task breakdown. The agent will ask 2-4 focused questions at a time about:
- **Scope**: What's included/excluded, edge cases, MVP vs nice-to-haves
- **Constraints**: Performance requirements, dependencies, compatibility
- **Preferences**: Approach/patterns, integration with existing code, testing expectations
- **Acceptance criteria**: How we know it's done, what success looks like

### zagi agent run

Executes the RALPH loop to automatically complete pending tasks.

```bash
# Run until all tasks complete (or fail 3x)
zagi agent run

# Run only one task then exit
zagi agent run --once

# Preview what would run without executing
zagi agent run --dry-run

# Set delay between tasks (default: 2 seconds)
zagi agent run --delay 5

# Safety limit - stop after N tasks
zagi agent run --max-tasks 10
```

The run loop will:
1. Pick the next pending task
2. Execute it with the configured agent
3. Mark it done on success (agent calls `zagi tasks done <task-id>`)
4. Skip tasks that fail 3 consecutive times
5. Continue until all tasks complete

### Executor Configuration

Control which AI agent executes tasks using environment variables:

```bash
# Use Claude Code (default)
ZAGI_AGENT=claude zagi agent run

# Use opencode
ZAGI_AGENT=opencode zagi agent run

# Use custom command with auto mode flags
ZAGI_AGENT=claude ZAGI_AGENT_CMD="myclaude --flag" zagi agent run

# Use completely custom tool (no auto flags)
ZAGI_AGENT_CMD="aider --yes" zagi agent run
```

See [docs/setup.md](docs/setup.md) for full configuration details.

### Blocked Commands (in agent mode)

These commands cause unrecoverable data loss and are blocked when `ZAGI_AGENT` is set:

| Command | Reason |
|---------|--------|
| `reset --hard` | Discards all uncommitted changes |
| `checkout .` | Discards all working tree changes |
| `clean -f/-fd/-fx` | Permanently deletes untracked files |
| `restore .` | Discards all working tree changes |
| `restore --worktree` | Discards working tree changes |
| `push --force/-f` | Overwrites remote history |
| `push --delete` | Deletes remote branch |
| `push <remote> :<branch>` | Deletes remote branch (refspec syntax) |
| `stash drop` | Permanently deletes stashed changes |
| `stash clear` | Permanently deletes all stashes |
| `branch -D` | Force deletes branch (even if not merged) |

Safe alternatives:
- `reset --soft` - keeps changes staged

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattzcarey/zagi](https://github.com/mattzcarey/zagi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
