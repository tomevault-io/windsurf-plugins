---
trigger: always_on
description: Comprehensive superskill consolidating 41 professional development skills across planning, testing, debugging, code review, git workflow, writing, architecture, meta-skills, thinking frameworks, and communication. Use when you need a complete reference for software development best practices, workflows, and methodologies.
---


# Professional Development Superskill

A comprehensive reference consolidating 41 professional skills into one complete guide.

## 📚 Table of Contents

### I. Development Process
1. [Brainstorming](#1-brainstorming) - Ideas → Designs
2. [Writing Plans](#2-writing-plans) - Implementation planning
3. [Executing Plans](#3-executing-plans) - Systematic execution
4. [Verification Before Completion](#4-verification-before-completion) - Quality gates

### II. Testing
5. [Test-Driven Development](#5-test-driven-development) - Tests first
6. [Testing with Subagents](#6-testing-with-subagents) - Multi-agent testing
7. [Testing Anti-Patterns](#7-testing-anti-patterns) - Mistakes to avoid
8. [Condition-Based Waiting](#8-condition-based-waiting) - Replace timeouts
9. [Test Under Pressure](#9-test-under-pressure) - Time-constrained testing

### III. Debugging
10. [Systematic Debugging](#10-systematic-debugging) - Four-phase framework
11. [Root Cause Tracing](#11-root-cause-tracing) - Find origins
12. [When Stuck](#12-when-stuck) - Get unstuck

### IV. Code Review
13. [Code Reviewer](#13-code-reviewer) - Reviewing effectively
14. [Requesting Reviews](#14-requesting-reviews) - Get good reviews
15. [Receiving Reviews](#15-receiving-reviews) - Handle feedback

### V. Git & Workflow
16. [Using Git Worktrees](#16-using-git-worktrees) - Multiple branches
17. [Finishing Branches](#17-finishing-branches) - Complete work

### VI. Writing & Documentation
18. [Writing Skills](#18-writing-skills) - Technical writing
19. [Writing Clearly and Concisely](#19-writing-clearly-and-concisely) - Clear prose
20. [Elements of Style](#20-elements-of-style) - Timeless principles

### VII. Architecture & Design
21. [Defense in Depth](#21-defense-in-depth) - Layered validation
22. [Subagent-Driven Development](#22-subagent-driven-development) - Autonomous agents
23. [Dispatching Parallel Agents](#23-dispatching-parallel-agents) - Coordination
24. [Collision Zone Thinking](#24-collision-zone-thinking) - Identify conflicts
25. [Preserving Productive Tensions](#25-preserving-productive-tensions) - Balance forces
26. [Simplification Cascades](#26-simplification-cascades) - Progressive simplification

### VIII. Meta-Skills
27. [Using Skills](#27-using-skills) - Apply skills effectively
28. [Using Superpowers](#28-using-superpowers) - Advanced techniques
29. [Sharing Skills](#29-sharing-skills) - Distribute knowledge
30. [Gardening Skills Wiki](#30-gardening-skills-wiki) - Maintain library
31. [Pulling Updates](#31-pulling-updates) - Sync repositories

### IX. Thinking & Analysis
32. [Meta-Pattern Recognition](#32-meta-pattern-recognition) - Cross-domain patterns
33. [Inversion Exercise](#33-inversion-exercise) - Think backwards
34. [Tracing Knowledge Lineages](#34-tracing-knowledge-lineages) - Knowledge evolution
35. [Search Agent](#35-search-agent) - Effective searching
36. [Remembering Conversations](#36-remembering-conversations) - Context retention

### X. Communication
37. [Persuasion Principles](#37-persuasion-principles) - Influence effectively
38. [Scale Game](#38-scale-game) - Communication at scale

---

# I. Development Process

## 1. Brainstorming

**Purpose:** Transform rough ideas into fully-formed designs through structured questioning.

**Core Principle:** Ask questions to understand, explore alternatives, present design incrementally for validation.

### The 6-Phase Process

1. **Understanding** - Ask ONE question at a time, gather purpose/constraints/criteria
2. **Exploration** - Propose 2-3 approaches with trade-offs
3. **Design Presentation** - Present in 200-300 word sections, validate each
4. **Design Documentation** - Write to `docs/plans/YYYY-MM-DD-<topic>-design.md`
5. **Worktree Setup** - Set up isolated workspace (if implementing)
6. **Planning Handoff** - Create implementation plan

### Key Principles
- **One question at a time** - Never overwhelm with multiple questions
- **YAGNI ruthlessly** - Remove unnecessary features
- **Explore alternatives** - Always propose 2-3 approaches
- **Incremental validation** - Validate each section
- **Flexible progression** - Go backward when needed

### When to Use
- Before writing code
- Before creating implementation plans
- When refining rough ideas into designs

---

## 2. Writing Plans

**Purpose:** Create detailed implementation plans from validated designs.

**Core Principle:** Break work into concrete, verifiable tasks with clear dependencies.

### Plan Structure

```markdown
# Implementation Plan: [Feature Name]

## Overview
- Goal: What we're building
- Context: Why we're building it
- Success criteria: How we know it's done

## Tasks

### Phase 1: Foundation
- [ ] Task 1 (Est: 2h)
  - Why: Reason for task
  - Acceptance: How to verify
  - Dependencies: What must be done first

### Phase 2: Core Features
...

## Risks & Mitigation
- Risk 1: Description → Mitigation strategy

## Testing Strategy
How will we verify this works?
```

### Key Elements
- **Task hierarchy** - Organize by phases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertpelloni/workspace](https://github.com/robertpelloni/workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
