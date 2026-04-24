---
trigger: always_on
description: This is an **AI-powered code review tool** that uses configurable agents to audit codebases for security, correctness, performance, and style issues.
---

# AI Code Reviewer - Agent Instructions

This is an **AI-powered code review tool** that uses configurable agents to audit codebases for security, correctness, performance, and style issues.

## Quick Start for AI Agents

**You are a code review agent.** Your configuration is defined in Oracle Agent Spec format.

### Available Agents

Load your agent configuration from `personas/<agent-name>/agent.yaml`:

| Agent | Focus | Use When |
|-------|-------|----------|
| `freebsd-angry-ai` | Security, style(9), POSIX | Production audits, security-critical code |
| `security-hawk` | Vulnerabilities, exploits | Security audits, penetration testing |
| `performance-cop` | Speed, algorithms, cache | Performance optimization |
| `friendly-mentor` | Learning, best practices | Training, onboarding, open source |
| `example` | Balanced, educational | General code review |

### Loading Your Configuration

```bash
# Validate agent configuration
python3 persona_validator.py personas/security-hawk

# Your system prompt is in:
cat personas/security-hawk/agent.yaml | grep -A1000 "system_prompt:"
```

### Agent Spec Format

Each agent is defined in [Oracle Agent Spec](https://oracle.github.io/agent-spec/26.1.0/) format:

```yaml
component_type: Agent
agentspec_version: "26.1.0"
name: "Agent Name"
description: "What this agent does"
system_prompt: |
  Your complete instructions and personality...
inputs:
  - title: "codebase_path"
    type: "string"
outputs:
  - title: "review_summary"
    type: "string"
```

## Project Structure

```
ai-code-reviewer/
├── personas/                 # Agent configurations
│   ├── freebsd-angry-ai/
│   │   └── agent.yaml       # Agent Spec configuration
│   ├── security-hawk/
│   ├── performance-cop/
│   ├── friendly-mentor/
│   └── example/
├── reviewer.py              # Main review loop
├── persona_validator.py     # Validates Agent Spec format
├── config.yaml.sample     # Configuration template
└── AGENTS.md               # This file
```

## Running Code Reviews

```bash
# Copy and configure
cp config.yaml.sample config.yaml
# Edit config.yaml: set source.root, source.build_command, review.persona

# Run review
python3 reviewer.py --config config.yaml

# Run forever mode (continuous review)
python3 reviewer.py --config config.yaml --forever
```

## Issue Tracking with bd (beads)

This project uses **bd (beads)** for issue tracking.

```bash
bd ready              # Find available work
bd show <id>          # View issue details  
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

### Workflow

1. `bd ready` - Check for unblocked issues
2. `bd update <id> --status in_progress` - Claim work
3. Implement, test, document
4. `bd close <id> --reason "Done"` - Complete work

## Session Completion

**Work is NOT complete until `git push` succeeds.**

```bash
# Before ending session:
git pull --rebase
bd sync
git push
git status  # MUST show "up to date with origin"
```

## Quality Gates

Before committing code changes:

```bash
# Run tests
python3 -m unittest discover -s tests -p "test_*.py"

# Validate agents
python3 persona_validator.py personas/freebsd-angry-ai
```

## Creating New Agents

1. Copy an existing agent:
   ```bash
   cp -r personas/example personas/my-agent
   ```

2. Edit `personas/my-agent/agent.yaml`:
   - Set `name`, `description`
   - Define `system_prompt` with personality and instructions
   - Configure `inputs`, `outputs`, `metadata`

3. Validate:
   ```bash
   python3 persona_validator.py personas/my-agent
   ```

4. Use in config.yaml:
   ```yaml
   review:
     persona: "personas/my-agent"
   ```

## References

- [Oracle Agent Spec](https://oracle.github.io/agent-spec/26.1.0/)
- [README.md](README.md) - Full documentation
- [SETUP_GUIDE.md](SETUP_GUIDE.md) - Installation guide

<!-- BEGIN BEADS INTEGRATION -->
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Dolt-powered version control with native sync
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**

```bash
bd ready --json
```

**Create new issues:**

```bash
bd create "Issue title" --description="Detailed context" -t bug|feature|task -p 0-4 --json
bd create "Issue title" --description="What this issue is about" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**

```bash
bd update <id> --claim --json
bd update bd-42 --priority 1 --json
```

**Complete work:**

```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanhubbard/ai-code-reviewer](https://github.com/jordanhubbard/ai-code-reviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
