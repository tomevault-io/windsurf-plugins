---
trigger: always_on
description: **Last Updated**: January 1, 2026
---

# File Naming Conventions

**Version**: 2.0  
**Last Updated**: January 1, 2026  
**Priority**: MANDATORY - All new content must follow these conventions

---

## 🎯 Core Principle

**Files should represent concepts.  
Folders should represent structure.  
Numbers should represent ordering — sparingly.**

If you remember only one line, remember this.

---

## ❌ Anti-Patterns (What NOT to Do)

### ❌ Do NOT encode hierarchy in filenames

**Avoid**:
```
Topic-Part1-A.md
Topic-Part1-B.md
Topic-Part1-A-A.md
Topic-Part1-B-A.md
```

**Why this is wrong**:
- Filenames become brittle and ugly
- `A/B/C` carries no semantic meaning
- Refactoring becomes painful
- URLs are hard to read
- GitHub navigation is confusing

### ❌ Avoid arbitrary "Part" unless it's a published volume

**Avoid**:
```
Glossary-Part1.md
Glossary-Part2.md
```

**Why this is wrong**:
- "Part" is an editorial accident, not a concept
- No semantic meaning for readers
- Hard to maintain and reorganize

### ❌ Avoid mixing sequence, hierarchy, and versioning

**Avoid**:
```
03_Architecture-Patterns-Part1-A-A.md
03_Architecture-Patterns-Part1-B-B.md
```

**Why this is wrong**:
- Confuses sequence with structure
- Makes it impossible to understand relationships
- Breaks when content is reorganized

---

## ✅ Preferred Patterns

### Pattern 1: Semantic Names (Best for Most Cases)

**Use semantic names that describe the content**:

```
evaluation-prep/
├── glossary/
│   ├── README.md
│   ├── core-concepts.md
│   ├── system-design.md
│   ├── cloud-architecture.md
│   ├── data-platforms.md
│   └── security.md
```

**Why this works**:
- Each file has a clear semantic scope
- Easy to add, split, or merge
- GitHub renders `README.md` automatically
- URLs are clean and stable
- Recruiter-friendly and maintainer-friendly

### Pattern 2: Ordered Files (Only When Sequence Matters)

**Use numbered prefixes ONLY when there's a deliberate learning order**:

```
evaluation-prep/
├── glossary/
│   ├── 01_fundamentals.md
│   ├── 02_system-design.md
│   ├── 03_cloud.md
│   └── 04_data.md
```

**Use this ONLY if**:
- There is a deliberate reading order
- You control the learning flow
- Sequence is part of the educational design

**Otherwise, use semantic names (Pattern 1)**.

### Pattern 3: Folder-Based Structure (Best for Growth)

**When content naturally groups, use folders**:

```
evaluation-prep/
├── glossary/
│   ├── README.md        # Index + navigation
│   ├── core-concepts.md
│   ├── system-design.md
│   └── cloud/
│       ├── README.md
│       ├── azure-fundamentals.md
│       ├── aws-fundamentals.md
│       └── multi-cloud.md
```

**Why this works**:
- Structure is clear and navigable
- Easy to expand without renaming
- GitHub navigation is intuitive
- Maintains clean URLs

---

## 📋 Naming Rules

### Rule 1: Use Semantic Names

**✅ Good**:
- `core-concepts.md`
- `system-design.md`
- `azure-fundamentals.md`
- `evaluation-questions.md`

**❌ Bad**:
- `Part1-A.md`
- `Part1-B.md`
- `Glossary-Part1.md`

### Rule 2: Use Hyphens for Multi-Word Names

**✅ Good**:
- `system-design.md`
- `azure-fundamentals.md`
- `evaluation-questions.md`

**❌ Bad**:
- `system_design.md` (underscores)
- `systemDesign.md` (camelCase)
- `System Design.md` (spaces)

### Rule 3: Use Numbers Sparingly

**Use numbers ONLY when**:
- There's a deliberate learning sequence
- Order matters for comprehension
- You're certain the order won't change

**✅ Good** (when sequence matters):
- `01_fundamentals.md`
- `02_advanced-concepts.md`
- `03_practical-applications.md`

**❌ Bad** (arbitrary numbering):
- `01_glossary.md` (if order doesn't matter)
- `02_questions.md` (if order doesn't matter)

### Rule 4: Use Folders for Structure

**When you need hierarchy, use folders, not filenames**:

**✅ Good**:
```
evaluation-prep/
├── questions/
│   ├── fundamentals.md
│   ├── system-design.md
│   └── cloud.md
```

**❌ Bad**:
```
evaluation-prep/
├── questions-fundamentals.md
├── questions-system-design.md
└── questions-cloud.md
```

---

## 🔄 When to Split Files

### Decision Framework

**Question 1**: Does the content exceed 150 lines?
- **Yes** → Continue to Question 2
- **No** → Keep as single file

**Question 2**: Can the content be split into distinct semantic concepts?
- **Yes** → Split into separate files with semantic names
- **No** → Continue to Question 3

**Question 3**: Is there a natural learning progression?
- **Yes** → Use numbered prefixes: `01_fundamentals.md`, `02_advanced.md`
- **No** → Use semantic names: `fundamentals.md`, `advanced.md`

**Question 4**: Will these concepts be expanded further?
- **Yes** → Create a folder structure
- **No** → Keep as separate files

### Example: Splitting a Large Glossary

**Before** (problematic):
```
01_Glossary-Part1-A.md  (150 lines)
01_Glossary-Part1-B.md  (150 lines)
01_Glossary-Part1-C.md  (150 lines)
```

**After** (semantic):
```
glossary/
├── README.md
├── core-concepts.md
├── system-design.md
└── cloud-architecture.md
```

**Or** (if sequence matters):
```
glossary/
├── README.md
├── 01_fundamentals.md
├── 02_system-design.md
└── 03_cloud-architecture.md
```

---

## 📁 Folder Structure Best Practices

### Use Folders When:

1. **Content naturally groups**:
   ```
   evaluation-prep/
   ├── questions/
   │   ├── fundamentals.md
   │   ├── system-design.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
