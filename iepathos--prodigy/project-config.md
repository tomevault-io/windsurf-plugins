---
trigger: always_on
description: This document contains Prodigy-specific documentation for Claude. General development guidelines are in `~/.claude/CLAUDE.md`.
---

# Prodigy Project Documentation

This document contains Prodigy-specific documentation for Claude. General development guidelines are in `~/.claude/CLAUDE.md`.

## Overview

Prodigy is a workflow orchestration tool that executes Claude commands through structured YAML workflows. It manages session state, tracks execution progress, and supports parallel execution through MapReduce patterns.

## Error Handling (Spec 101, 168)

### Core Rules
- **Production code**: Never use `unwrap()` or `panic!()` - use Result types and `?` operator
- **Test code**: May use `unwrap()` and `panic!()` for test failures
- **Static patterns**: Compile-time constants (like regex) may use `expect()`

### Error Types
- Storage: `StorageError`
- Worktree: `WorktreeError`
- Command execution: `CommandError`
- General: `anyhow::Error`

### Context Preservation
Prodigy uses Stillwater's `ContextError<E>` to preserve operation context through the call stack:

```rust
use prodigy::cook::error::ResultExt;

fn process_item(id: &str) -> Result<(), ContextError<ProcessError>> {
    create_worktree(id).with_context(|| format!("Creating worktree for {}", id))?;
    execute_commands(id).context("Executing commands")?;
    Ok(())
}
```

**Benefits**: Full context trail in error messages, DLQ integration, zero runtime overhead on success path.

## Claude Observability (Spec 121)

### JSON Log Tracking
Claude Code creates detailed JSON logs at `~/.local/state/claude/logs/session-{id}.json` containing:
- Complete message history and tool invocations
- Token usage and session metadata
- Error details and stack traces

**Access logs:**
- Verbose mode: `prodigy run workflow.yml -v` shows log path after each command
- Programmatically: `result.json_log_location()`
- MapReduce events: `AgentCompleted` and `AgentFailed` include `json_log_location`
- DLQ items: `FailureDetail` preserves log location

**Debug failed agents:**
```bash
# Get log path from DLQ
prodigy dlq show <job_id> | jq '.items[].failure_history[].json_log_location'

# Inspect the log
cat /path/to/log.json | jq '.messages[-3:]'
```

## Custom Merge Workflows

Define custom merge workflows with validation, testing, and conflict resolution:

```yaml
merge:
  commands:
    - shell: "git fetch origin && git merge origin/main"
    - shell: "cargo test && cargo clippy"
    - claude: "/prodigy-merge-worktree ${merge.source_branch} ${merge.target_branch}"
  timeout: 600
```

**Available variables:**
- `${merge.worktree}` - Worktree name
- `${merge.source_branch}` - Source branch (worktree)
- `${merge.target_branch}` - Target branch (original branch)
- `${merge.session_id}` - Session ID

**Streaming**: Use `-v` flag or set `PRODIGY_CLAUDE_CONSOLE_OUTPUT=true` for real-time output.

## MapReduce Workflows

### Basic Structure
```yaml
name: workflow-name
mode: mapreduce

setup:
  - shell: "generate-work-items.sh"

map:
  input: "items.json"
  json_path: "$.items[*]"
  max_parallel: 10

  agent_template:
    - claude: "/process '${item}'"
    - shell: "test ${item.path}"
      on_failure:
        claude: "/fix-issue '${item}'"

reduce:
  - claude: "/summarize ${map.results}"
  - shell: "echo 'Processed ${map.successful}/${map.total}'"
```

### Commit Validation (Spec 163)
Commands with `commit_required: true` enforce commit creation. Agent fails if no commit is made.

```yaml
agent_template:
  - shell: |
      echo "data" > file.txt
      git add file.txt
      git commit -m "Add data"
    commit_required: true
```

**Validation behavior:**
- HEAD SHA checked before/after command
- No new commits → agent fails with `CommitValidationFailed`
- Failed agents added to DLQ with full context
- Agents with commits: merged to parent; without commits: cleaned up

### Checkpoint & Resume (Spec 134)
Prodigy checkpoints all phases (setup, map, reduce) for recovery:

**Resume commands:**
```bash
prodigy resume <session-or-job-id>
prodigy resume-job <job_id>
```

**State preservation:**
- Setup: Checkpoint after completion
- Map: Checkpoint after configurable work items processed
- Reduce: Checkpoint after each command
- Variables, outputs, and agent results preserved

**Storage:** `~/.prodigy/state/{repo}/mapreduce/jobs/{job_id}/`

### Concurrent Resume Protection (Spec 140)
RAII-based locking prevents multiple resume processes:
- Exclusive lock acquired automatically before resume
- Lock released on completion or failure
- Stale locks (crashed processes) auto-detected and cleaned
- Lock files: `~/.prodigy/resume_locks/{id}.lock`

### Worktree Isolation (Spec 127)
All phases execute in isolated worktrees:

```
original_branch → parent worktree (session-xxx)
                  ├→ Setup executes here
                  ├→ Agent worktrees (branch from parent, merge back)
                  ├→ Reduce executes here
                  └→ User prompt: Merge to {original_branch}?
```

**Benefits:** Main repo untouched, parallel execution, full isolation, user-controlled merge.

### Cleanup Handling (Spec 136)
Agent success independent of cleanup status:
- Successful agents preserved even if cleanup fails
- Orphaned worktrees registered: `~/.prodigy/orphaned_worktrees/{repo}/{job_id}.json`
- Clean orphaned: `prodigy worktree clean-orphaned <job_id>`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iepathos/prodigy](https://github.com/iepathos/prodigy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
