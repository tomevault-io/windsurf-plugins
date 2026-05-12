---
trigger: always_on
description: Clawrium is a CLI tool (`clm`) that manages AI agent fleets across your local network. Point it at any machine, and it handles deployment, configuration, and lifecycle management via SSH and Ansible.
---

# Clawrium - An aquarium for *claws

## How It Works

Clawrium is a CLI tool (`clm`) that manages AI agent fleets across your local network. Point it at any machine, and it handles deployment, configuration, and lifecycle management via SSH and Ansible.

```
Your Machine (clm CLI)
    │
    ├── Host A ──> zeroclaw instance
    ├── Host B ──> openclaw instance
    └── Host C ──> nemoclaw instance, zeroclaw instance
```

## Why

- **Single pane of glass**: Manage all agents from one CLI instead of SSH-ing to each host
- **Consistent lifecycle**: Same commands for install, configure, start, stop, remove across all agent types
- **Secrets management**: Secure API key storage with per-agent isolation
- **Fleet visibility**: `clm ps` shows status of all agents across all hosts

## Who Is This For

- **Homelabbers**: Run multiple AI assistants on spare hardware
- **Teams**: Standardize agent deployment across developer machines
- **Experimenters**: Try different models/agents without manual setup on each host

## Quickstart

```bash
# Install
uv tool install clawrium

# Add a host
clm host init 192.168.1.100 --user myuser
clm host add 192.168.1.100 --alias mybox

# Install an agent
clm agent install --type openclaw --host mybox

# Configure and start
clm agent configure <agent-name>
clm agent start <agent-name>

# Check fleet status
clm ps
```

## Key Concepts

- **Host**: A machine in your network that runs one or more agents
- **Agent**: An AI assistant instance (zeroclaw, nemoclaw, or openclaw)
- **Agent Type**: The specific AI assistant implementation (e.g., zeroclaw, nemoclaw, openclaw)
- **Agent Name**: The unique identifier for an installed agent instance
- **Registry**: Platform-defined agent types with versions, dependencies, and templates

## Resources

- Repository: https://github.com/ric03uec/clawrium
- Project Board: https://github.com/users/ric03uec/projects/1
- Version: 26.4.7

## Tech Stack

- **CLI**: Python + Typer
- **Execution**: ansible-runner
- **Packaging**: uv/uvx
- **User Data**: `~/.config/clawrium/`

## Development

Always use `make` commands to run tests and validate changes:

```bash
make test       # Run tests (required before commits)
make lint       # Check code style
make format     # Format code
make test-cov   # Run tests with coverage
```

## Development Workflow

GitHub Issues are the single source of truth. See [CONTRIBUTING.md](CONTRIBUTING.md) for full workflow documentation.

### Worktree Convention

For parallel issue execution, use git worktrees with this naming:

```
<repo-parent>/<repo-name>-issue-<number>/
```

Example:
```
~/projects/clawrium/           # Main repo
~/projects/clawrium-issue-35/  # Worktree for issue 35
```

Trigger with: `/itx:execute 35 in a subtree` or `/itx:execute 35 --worktree`

### Quick Reference

```
New Issue → /itx:triage → /itx:plan-create → /itx:plan-scaffold → /itx:execute → /itx:verify → /itx:review-pr → Merge
```

### Skills

| Skill | Purpose |
|-------|---------|
| `/itx:bug-new` | Create bug issue (asks for customer outcome) |
| `/itx:issue-new` | Create feature issue (asks for customer outcome) |
| `/itx:triage` | Review unlabeled issues |
| `/itx:plan-create <n>` | Create high-level implementation plan |
| `/itx:plan-scaffold <n>` | Create phased execution with entry/exit criteria |
| `/itx:execute <n>` | Execute issue (parent or subtask) |
| `/itx:verify` | Run tests and lint |
| `/itx:review-pr [n]` | Review PR (MCP or manual) |

### Task-Based Execution

The `/itx:execute` skill uses a structured task checklist approach to prevent getting lost during execution:

**Planning Phase (Mandatory)**:
1. Read implementation plan from issue
2. Create implementation tasks using `TaskCreate()` for each phase/step
3. Create verification tasks (tests, lint, review if MCP enabled)
4. Set dependencies between tasks if needed
5. Review task list to confirm structure

**Execution Phase**:
1. Get next pending task using `TaskList()`
2. Mark task `in_progress` using `TaskUpdate()`
3. Execute the task requirements
4. Mark task `completed`
5. Check progress with `TaskList()`
6. Repeat until all tasks done

**Example Task Creation**:
```python
# Implementation task
TaskCreate(
    subject="Implement: Update CLI help text",
    description="Update all help text in src/clawrium/cli/agent.py",
    activeForm="Updating CLI help text"
)

# Verification task
TaskCreate(
    subject="Run test suite",
    description="Execute 'make test' and ensure all tests pass",
    activeForm="Running tests"
)
```

**Recovery Mechanism**:
If execution feels unclear or you lose orientation:
- Run `TaskList()` to see current state
- Check which task is `in_progress`
- Review that task's description
- Complete current task before starting next

## Review

Review requirements depend on whether MCP-based automated review is configured in `.claude/itx-config.json`.

### Check Review Mode

```bash
ITX_CONFIG="$(git rev-parse --show-toplevel)/.claude/itx-config.json"
if [ -f "$ITX_CONFIG" ]; then
  REVIEW_ENABLED=$(jq -r '.mcp.review_enabled // false' "$ITX_CONFIG")
else
  REVIEW_ENABLED="false"
fi
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ric03uec/clawrium](https://github.com/ric03uec/clawrium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
