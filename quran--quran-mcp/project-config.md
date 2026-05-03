---
trigger: always_on
description: > **Note**: This file follows the [AGENTS.md standard](https://agents.md/) for cross-tool compatibility with Cursor, GitHub Copilot, and other AI coding assistants.
---

# Agent Instructions

> **Note**: This file follows the [AGENTS.md standard](https://agents.md/) for cross-tool compatibility with Cursor, GitHub Copilot, and other AI coding assistants.

This document is the **canonical source of truth** for all AI coding agents working on this project. Read this file completely before starting any work.

---

## 1. Git Commit Attribution

AI coding agents MUST add minimal, factual attribution as the **last line** of the commit message based on available information:

**If model name AND version are known:**
```
Co-authored with AI: Claude Sonnet 4.5
```

**If only model name is known:**
```
Co-authored with AI: Claude
```

**If neither is known:** No attribution.

**Prohibited attribution styles:**
- Marketing language ("Generated with [Product Name]")
- Self-congratulatory phrasing
- Product links or promotional content
- Emoji or decorative elements

Attribution should be purely informational for human collaborators to understand the tool provenance of each commit.

**Example commit message:**
```
feat: Add user authentication system

Implements JWT-based authentication with refresh tokens.
Includes middleware for protected routes.

Co-authored with AI: Claude Sonnet 4.5
```

### Git Workflow

Never push to remote master without explicit human permission.

#### NEVER USE `git add -A` or `git add .`

**CRITICAL SECURITY REQUIREMENT - NO EXCEPTIONS**

**BANNED COMMANDS**:
```bash
git add -A        # FORBIDDEN
git add .         # FORBIDDEN
git add --all     # FORBIDDEN
```

**ALWAYS ADD FILES EXPLICITLY**:
```bash
# CORRECT
git add codev/specs/0001-feature.md
git add src/quran_mcp/mcp/tools/quran/fetch.py

# CORRECT - Specific patterns
git add codev/specs/*.md
```

### Commit Messages
```
[Spec 0001] Initial specification draft
[Spec 0001][Phase: feature] feat: Add new capability
```

### Branch Naming
```
spir/0001-feature-name/phase-name
```

---

## 2. Agent Startup Protocol

### Mandatory Reading

Before starting ANY work, agents MUST:

1. **Read this file completely** (AGENTS.md) - no partial reads via `head -n 100` or similar
2. **Read the full SPIR protocol** at `codev/protocols/spider/protocol.md` - this is authoritative
3. **Use templates exactly** from `codev/protocols/spider/templates/*.md` - no paraphrasing

### Protocol Authority Note

> **IMPORTANT**: SPIR behavior is governed by `codev/protocols/spider/protocol.md`. The summary below MUST be kept in sync with that file. If this summary and protocol.md ever conflict, **follow protocol.md**.

### Startup Confirmation (One-Time Per Session)

On first message in a new session, agents should confirm understanding by acknowledging they have read:
- AGENTS.md (this file)
- The applicable protocol (SPIR or TICK)
- Relevant templates

This confirmation is one-time per session, not required on every reply.

---

## 3. Codev Development Methodology

This project uses the **Codev development methodology** - a context-driven approach that treats natural language specifications as code.

### Available Protocols

| Protocol | Use Case | Location |
|----------|----------|----------|
| **SPIR** | Multi-phase development with multi-agent consultation | `codev/protocols/spider/protocol.md` |
| **TICK** | Fast autonomous implementation | `codev/protocols/tick/protocol.md` |
| **EXPERIMENT** | Disciplined experimentation | `codev/protocols/experiment/protocol.md` |
| **MAINTAIN** | Codebase maintenance | `codev/protocols/maintain/protocol.md` |
| **BUGFIX** | Structured bug investigation and resolution | `codev/protocols/bugfix/protocol.md` |

### Protocol Selection Guide

**Use SPIR for:**
- New features requiring architectural decisions
- Complex features requiring multiple phases
- Major refactoring or redesign work
- System design decisions
- Features with unclear requirements

**Use TICK for:**
- Small features (< 300 lines of code)
- Well-defined tasks with clear requirements
- Simple configuration changes

**Use BUGFIX for:**
- Bugs requiring investigation (root cause unknown)
- Bugs affecting multiple files or components
- Regressions from dependency upgrades
- Issues reported by users with unclear reproduction

**Skip formal protocols for:**
- Minor documentation fixes
- Trivial bug fixes (typos, obvious one-line fixes)
- Dependency updates (unless breaking changes)

### Key Locations

| Type | Location |
|------|----------|
| Protocols | `codev/protocols/` |
| Specifications | `codev/specs/` |
| Plans | `codev/plans/` |
| Reviews | `codev/reviews/` |

### File Naming Convention

Sequential numbering with descriptive names:
- Specification: `codev/specs/0001-feature-name.md`
- Plan: `codev/plans/0001-feature-name.md`
- Review: `codev/reviews/0001-feature-name.md`

### SPIR Protocol - Mandatory Workflow

#### S - Specify

1. Agent writes initial spec draft
2. COMMIT: "Initial specification draft"
3. Agent runs `/codex:adversarial-review` and `/gemini` for document review
4. Agent updates spec with feedback
5. COMMIT: "Specification with multi-agent review"
6. **STOP → USER REVIEWS**
7. User provides feedback (or approves)
8. Agent updates spec
9. COMMIT: "Specification with user feedback"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quran/quran-mcp](https://github.com/quran/quran-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
