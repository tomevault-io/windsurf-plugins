---
trigger: always_on
description: > **Give your AI the memory it deserves.**
---

# Intent-Engine: AI Long-Term Memory System

> **Give your AI the memory it deserves.**

---

## The Challenge

AI assistants are incredibly capable within a single conversation. But when sessions end:

```
Day 1: "Let's build authentication"
       AI works brilliantly, makes smart decisions...
       [session ends]

Day 2: "Continue authentication"
       AI: "What authentication? I have no memory of this."
```

**The challenge isn't capability—it's continuity.**

User tasks span days, weeks, months. AI conversations are ephemeral.

---

## The Solution: External Memory

Intent-Engine gives AI persistent memory across sessions:

| Challenge | Solution | How ie Helps |
|-----------|----------|--------------|
| Sessions are ephemeral | Persistent task state | `ie status` restores full context |
| Decisions get lost | Decision history | `ie log` records the "why" |
| Context needs repeating | External memory | `ie search` retrieves past work |
| Complex work needs structure | Hierarchical tasks | `ie plan` organizes intent |

**One command restores everything:**

```bash
ie status
# Returns: current task, goal, approach, decision history, subtasks
```

---

## 6 Principles of Effective AI Collaboration

### 1. Intent Anchoring

> Clear goals lead to better outcomes.

```
Vague:                          Clear:
"I'm helping you code..."       "I'm on #42: Implement JWT auth"
(What code? Why? Goal?)         "Goal: Users access protected resources"
                                "Current: Implementing validation middleware"
```

**When AI knows exactly what it's working on, it stays focused.**

### 2. Decomposition = Understanding

> Breaking down problems reveals their structure.

```
Good decomposition reveals:
├─ Dependencies between components
├─ Where key decisions lie
├─ Risks and uncertainties
└─ True work distribution
```

**If you can decompose it well, you understand it well.**

### 3. Decision Transparency

> Record the "why" behind every choice.

```
Two weeks later, user asks: "Why HS256?"

Without record:
  AI: "...I don't know"

With record:
  AI: checks history → "Single-app scenario, asymmetric encryption adds unnecessary complexity"
```

**Decision logs are messages to your future self.**

### 4. Focus Discipline

> One thing at a time. Finish before moving on.

```
Scattered:                      Focused:
"Working on" 5 things           Current focus: #42
Each half-done                  Depth-first: complete subtasks first
Easy to get lost                Report blocks: log when stuck
```

**Focus isn't "I'm looking at" - it's "I commit to complete".**

### 5. Verifiable Completion

> Know what "done" looks like before you start.

```
Vague:                          Clear:
"Implement user auth"           Completion criteria:
 ↓                              1. POST /auth/login returns JWT
"I implemented it" (really?)    2. Protected routes validate token
                                3. Expired token returns 401
                                4. All tests pass
```

**Clear criteria prevent premature completion claims.**

### 6. Context Recovery

> Externalize everything needed to continue later.

```
Session recovery test:
  If this session ends now, can I:
  1. Know what we were doing?
  2. Find relevant decisions?
  3. Continue without user repeating everything?
```

**If yes, the context is properly externalized.**

---

## What ie Really Is

```
TodoWrite = Sticky notes     ie = External brain
Use and discard              Persistent memory
Single session               Cross-session
No structure                 Hierarchical
No history                   Traceable
```

### Command Meanings

| Command | Function | Purpose |
|---------|----------|---------|
| `ie status` | View current state | **Context recovery** - restore working memory |
| `ie plan` | Batch create/update tasks | **Intent persistence** - externalize goals (batch) |
| `ie task create` | Create a single task | **Task creation** - with metadata, deps, owner |
| `ie task get` | Get task details | **Task inspection** - with events and context |
| `ie task update` | Update a task | **Task mutation** - any field, metadata, deps |
| `ie task list` | List/filter tasks | **Task discovery** - filter, sort, tree view |
| `ie task delete` | Delete a task | **Task cleanup** - with optional cascade |
| `ie task start` | Start a task | **Focus + status** - sets doing and focuses |
| `ie task done` | Complete a task | **Completion** - by ID or current focus |
| `ie task next` | Suggest next task | **Prioritization** - context-aware pick |
| `ie log` | Record events | **Decision history** - capture the "why" |
| `ie search` | Find history | **Memory retrieval** - access past context |

---

## Quick Decision: ie vs TodoWrite

| Scenario | TodoWrite | ie |
|----------|-----------|-----|
| Single session, disposable | ✓ | |
| Cross-session work | | ✓ |
| Need to record "why I decided this" | | ✓ |
| Complex multi-level breakdown | | ✓ |
| Need to review decision history | | ✓ |

**Simple rule:**
- Would be a shame to lose → ie
- Use once and discard → TodoWrite

**ie task vs ie plan:**
- Single task operation (create, update, delete) → `ie task` (preferred)
- Batch create/update multiple tasks at once → `ie plan`

---

## Task Execution Framework

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wayfind/intent-engine](https://github.com/wayfind/intent-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
