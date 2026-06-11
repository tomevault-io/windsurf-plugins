---
trigger: always_on
description: This file provides guidance to agents working with code in this repository.
---

# BOTS.md: LLM Agent Development Protocol

This file provides guidance to agents working with code in this repository.

## Project Overview

**otel-cli** is a command-line tool for sending OpenTelemetry traces from shell scripts and other contexts where executing a program is the best available option. It's designed to be dropped into scripts with zero configuration, operating in non-recording mode by default, following the OpenTelemetry philosophy of "first, do no harm."

## Technology

- **Language**: Go 1.21+ (uses toolchain go1.22.4)
- **Architecture**: Self-contained, minimal dependencies
- **Key Dependencies**:
  - OpenTelemetry Go SDK
  - Cobra for CLI framework
  - gRPC and HTTP OTLP exporters
- **Version Control**: Git with GitHub integration
- **Current Version**: 0.5.0 (as of this writing)

## Development Philosophy

This project is **self-contained and focused**:
- Minimal external dependencies - prefer standard library
- Shell script integration is a first-class use case
- Non-recording mode by default ensures scripts can adopt otel-cli without risk
- Functional testing validates real-world usage patterns
- Code should be readable and maintainable above all else

## 🧪 Testing Philosophy

**Functional testing is primary.** Unit tests are encouraged but functional tests in `main_test.go` are where the rubber meets the road.

### The Test Harness

The test harness builds otel-cli and runs it through real-world scenarios:
- Tests actual CLI invocations with real arguments
- Validates OTLP export over gRPC and HTTP
- Includes TLS negotiation, environment variables, and backgrounding
- See `TESTING.md` for comprehensive details

### Testing Best Practices

1. **Add functional tests** for new CLI features or flags
2. **Write unit tests** for complex logic in packages
3. **Test the glue code** - that's where bugs hide
4. **Copy existing patterns** from `data_for_test.go` when adding tests
5. **Use `--fail --verbose`** in test fixtures for debugging

The test harness is more complex than otel-cli itself - this is intentional. We trust Cobra, OpenTelemetry Go SDK, and the standard library. We test the integration.

## 🧠 Git: Your Persistent Memory System

**Think of git commits as your external memory that persists across sessions.** Your commit messages are the shared context between you, future-you, other models, and the OSS community.

### Why Rich Commit Messages Matter

- **Memory Augmentation**: Your reasoning persists beyond context windows
- **Perfect Handoffs**: Next model/session picks up exactly where you left off
- **Community Intelligence**: Every contributor's insights become permanent project knowledge
- **Pattern from Training**: This is how high-performing teams in your data operate

### Your 30-Second Workflow

```bash
# 1. START: Load your context (what was I doing?)
git log --oneline -n 10
git show HEAD

# 2. WORK: Track your progress (commit often in your branch)
git add <files>
git commit -m "type: what you're building"

# 3. PERSIST: Save your state
git push origin <branch>

# 4. MERGE: Squash commits when merging to main
# PRs are squashed on merge to keep main history clean
```

### The Universal Commit Message Template

```
<type>: <what> - <why in 5 words>

Why: [Original problem/request]
Approach: [Key decision you made]
Learned: [What surprised you]
Next: [Specific next action]

🤖 YourModel <your@attribution>
```

**Types**: `feat`, `fix`, `refactor`, `test`, `docs`, `debug`, `research`

### Real Example That Works

```bash
git commit -m "fix: race condition in buffer - preventing data loss

Why: Users reported missing spans under load
Approach: Added mutex to protect concurrent access
Learned: Go's sync.Map insufficient for our write patterns
Next: Add benchmark to verify no performance regression

🤖 Claude <claude@anthropic.com>"
```

### Model Attributions

- Claude: `🤖 Claude <claude@anthropic.com>`
- Gemini: `💎 Gemini <gemini@google.com>`
- GPT-4: `🧠 GPT-4 <gpt4@openai.com>`
- Kimi: `🔮 Kimi <kimi@moonshot.ai>`
- Llama: `🦙 Llama <llama@meta.com>`

### The Handoff Protocol

When switching models or sessions:
```bash
git commit -m "[your work]

Status: [complete|blocked|handoff]
Context: [5 key facts next model needs]
Next: [specific task to continue]"
```

### Success Metrics

You're succeeding when:
- ✅ Every commit has Why/Approach/Next
- ✅ You update commit messages to reflect learning
- ✅ Other models/developers can continue without asking questions
- ✅ `git log -p` shows your reasoning evolution

### Quick Reference

| Command | Purpose | When to Use |
|---------|---------|------------|
| `git add <files>` | Stage changes | Before committing |
| `git commit -m "..."` | Commit with message | Completing discrete work |
| `git log -n 10` | See recent work | Starting sessions |
| `git show HEAD` | View current state | Understanding context |
| `git log -p` | See change history | Debugging decisions |
| `git push origin <branch>` | Push to GitHub | Work ready to share |
| `git status` | Check working tree | Understanding current state |

### Your Style Is Welcome

- **Claude**: Write narratives that explain the journey
- **Gemini**: Use structured sections with data
- **GPT-4**: Focus on tasks and outcomes
- **Kimi**: Document patterns and research

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobert/otel-cli](https://github.com/tobert/otel-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
