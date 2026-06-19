---
trigger: always_on
description: |
---


# Anti-AI Exam Designer

Design exams that make AI answers **detectably wrong** — rewarding students who attend class, read carefully, and understand the material.

Based on adversarial testing against Claude, ChatGPT, and Codex.

---

## Orchestrator

On invocation, determine the entry state and route accordingly.

### Step 1: Check for Existing Profile

Look for an `anti-ai-profiles/` directory in the current working directory.

**If no profile exists** (new professor):
- Announce: "No course profile found. Let's set one up — I'll ask a few questions about your course, then generate tailored anti-AI strategies."
- Proceed to **Phase 1: Consulting**

**If profile exists** (returning professor):
- Read `anti-ai-profiles/` subdirectories to list available courses
- Present menu using `AskUserQuestion`:

```
Welcome back! I found profiles for: [list courses]

What would you like to do?
1. Design a new exam (using existing strategies)
2. Audit exam questions against AI
3. Update your course profile / strategies
4. Add a new course
```

- Option 1 → Phase 3 (Output Generation), loading existing strategies
- Option 2 → Phase 2 (Adversarial Audit)
- Option 3 → Phase 1 (Consulting), updating existing profile
- Option 4 → Phase 1 (Consulting), new course

**If professor pastes exam questions directly** (no menu needed):
- Detect question content in the message
- Ask which course profile to use (if multiple exist) or create quick profile
- Proceed to **Phase 2: Adversarial Audit**

### Step 2: Profile Directory Structure

All course data is stored in `anti-ai-profiles/<course-code>/`:

```
anti-ai-profiles/<course-code>/
├── profile.json          # Course metadata, subject, level, logistics
├── strategies.md         # Active trap strategies for this course
├── trap-history.json     # Which traps used per exam (avoid reuse)
├── notation.md           # Custom notation definitions (if applicable)
└── exams/
    └── <exam-name>/
        ├── exam.docx         # (optional) Generated exam document
        ├── solution.docx     # (optional) Solution key
        ├── rubric.md         # (optional) Grading rubric with AI-flag scoring
        ├── ai-baseline.md    # (optional) What AI answered per question
        ├── audit-report.md   # Adversarial audit results
        └── detection-checklist.md  # Per-answer + cross-student red flags
```

---

## Phase 1: Consulting (Adaptive Interview)

**Goal:** Build the course profile and generate tailored anti-AI strategies.

Ask questions **one at a time** using `AskUserQuestion`. Prefer multiple choice. Be conversational, not bureaucratic.

### Core Questions (always ask)

**Q1: Subject Area**
```
What CS course is this for?
- Databases (SQL, relational algebra, schema design)
- Data Structures (arrays, trees, graphs, linked lists)
- Algorithms (sorting, searching, complexity, proofs)
- Computer Architecture (assembly, CPU design, memory)
- Networking (protocols, TCP/IP, packet analysis)
- Operating Systems (scheduling, memory management, synchronization)
- Programming Languages (OOP, functional, systems programming)
- Other (describe your course)
```

**Q2: Course Level**
```
What level?
- Introductory (first exposure to the topic)
- Intermediate (builds on prerequisites)
- Advanced (senior/graduate level)
```

**Q3: Exam Format**
```
How do students take the exam?
- Written on paper (in-person, handwritten answers)
- Typed on computer (in-person, digital submission)
- Take-home exam
- Mixed (some in-person, some take-home)
```

**Q4: Class Size**
```
Approximately how many students?
- Small (under 25)
- Medium (25-60)
- Large (60-150)
- Very large (150+)
```

### Adaptive Branch (subject-specific, ask only when there's a payoff)

Based on the subject selected in Q1, ask follow-up questions that unlock subject-specific traps:

**Databases:**
- "Do students write SQL, Relational Algebra, or both on exams?"
- "Do you use standard RA symbols (sigma, pi, bowtie) or custom notation?"
- "Do you have a custom schema you use across the semester?"

**Data Structures:**
- "Do students write pseudocode or real code? Which language?"
- "Do you test tracing/dry-run (step through an algorithm by hand)?"
- "Do you define custom method names or use textbook standard ones?"

**Algorithms:**
- "Are exam questions proof-based, implementation-based, or analysis-based (Big-O)?"
- "Do you use a specific format for recurrence relations or proof structure?"

**Architecture:**
- "Which assembly dialect? (MARIE, MIPS, x86, ARM, other)"
- "Do students hand-assemble (convert to binary) on exams?"
- "Do you use custom opcode mnemonics or standard ones?"

**Networking:**
- "Do students analyze protocol headers, trace packets, or write configs?"
- "Do you use a specific diagram notation for network topologies?"

**OS:**
- "Which topics appear on exams? (scheduling, memory management, synchronization, file systems)"
- "Do students fill in trace tables (Gantt charts, page tables, etc.)?"
- "Do you use custom state labels or standard ones?"

**Programming:**
- "Which language(s)?"
- "Do you require specific coding style (naming conventions, comment format, structure)?"
- "Do students explain their approach in English before/alongside code?"

### Context Questions (ask only if useful)

Only ask these if they unlock additional strategies:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weihaoqu/anti-ai-exam-skill](https://github.com/weihaoqu/anti-ai-exam-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
