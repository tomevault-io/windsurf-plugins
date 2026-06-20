---
trigger: always_on
description: Documentation-first development methodology. The goal is AI-ready documentation - when docs are clear enough, code generation becomes automatic. Triggers on "Build", "Create", "Implement", "Document", or "Spec out". Version 3.4 adds complete 13-item Clarity Gate with scoring rubric and self-assessment.
---


# Stream Coding v3.4: Documentation-First Development

## ⚠️ CRITICAL REFRAME: THIS IS A DOCUMENTATION METHODOLOGY, NOT A CODING METHODOLOGY

**The Goal:** AI-ready documentation. When documentation is clear enough, code generation becomes automatic.

**The Insight:**

> "If your docs are good enough, AI writes the code. The hard work IS the documentation. Code is just the printout."

**v3.4 Core Addition:** Complete 13-item Clarity Gate with scoring rubric. The gate is the methodology—skip it and you're back to vibe coding.

---

## CHANGELOG

| Version | Changes                                                                                                                                                               |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3.0     | Initial Stream Coding methodology                                                                                                                                     |
| 3.1     | Clearer terminology, mandatory Clarity Gate                                                                                                                           |
| 3.3     | Document-type-aware placement (Anti-patterns, Test Cases, Error Handling in implementation docs)                                                                      |
| 3.3.1   | Corrected time allocation (40/40/20), added Phase 4, added Rule of Divergence                                                                                         |
| **3.4** | **Complete 13-item Clarity Gate, scoring rubric with weights, self-assessment questions, 4 mandatory section templates, Documentation Audit integrated into Phase 1** |

---

## THE STREAM CODING TRUTH

```
Messy Docs → Vague Specs → AI Guesses → Rework Cycles → 2-3x Velocity
Clear Docs → Clear Specs → AI Executes → Minimal Rework → 10-20x Velocity
```

**Why Most "AI-Assisted Development" Fails:**

- People feed AI messy docs
- AI generates code based on assumptions
- Code doesn't match intent
- Endless revision cycles
- Result: Marginally faster than manual coding

**Why Stream Coding Achieves 10-20x:**

- Documentation is clarified FIRST
- AI has zero ambiguity
- Code matches intent on first pass
- Minimal revision
- Result: Documentation time + automatic code generation

---

## DOCUMENT TYPE ARCHITECTURE

**The Rule:** Not all documents need all sections. Putting implementation details in strategic documents violates single-source-of-truth.

> "If AI has to decide where to find information, you've already lost velocity."

### Document Types

| Type               | Purpose      | Examples                                                   |
| ------------------ | ------------ | ---------------------------------------------------------- |
| **Strategic**      | WHAT and WHY | Master Blueprint, PRD, Vision docs, Business cases         |
| **Implementation** | HOW          | Technical Specs, API docs, Module specs, Architecture docs |
| **Reference**      | Lookup       | Schema Reference, Glossary, Configuration                  |

### Section Placement Matrix

| Section                      | Strategic Docs  | Implementation Docs | Reference Docs |
| ---------------------------- | --------------- | ------------------- | -------------- |
| **Deep Links (References)**  | ✅ Required     | ✅ Required         | ✅ Required    |
| **Anti-patterns**            | ❌ Pointer only | ✅ Required         | ❌ N/A         |
| **Test Case Specifications** | ❌ Pointer only | ✅ Required         | ❌ N/A         |
| **Error Handling Matrix**    | ❌ Pointer only | ✅ Required         | ❌ N/A         |

### Why This Matters

**Wrong (violates single-source-of-truth):**

```
Master Blueprint
├── Strategy content
├── Anti-patterns ← WRONG: duplicates Technical Spec
├── Test Cases ← WRONG: duplicates Testing doc
└── Error Matrix ← WRONG: duplicates Error Handling doc
```

**Right (single-source-of-truth):**

```
Master Blueprint (Strategic)
├── Strategy content
└── References
    └── Pointer: "Anti-patterns → Technical Spec, Section 7"

Technical Spec (Implementation)
├── Implementation details
├── Anti-patterns ← CORRECT: lives here
├── Test Cases ← CORRECT: lives here
└── Error Matrix ← CORRECT: lives here
```

---

## THE 4-PHASE METHODOLOGY

### Time Allocation

| Phase                           | Time | Focus                                               |
| ------------------------------- | ---- | --------------------------------------------------- |
| Phase 1: Strategic Thinking     | 40%  | WHAT to build, WHY it matters                       |
| Phase 2: AI-Ready Documentation | 40%  | HOW to build (specs so clear AI has zero decisions) |
| Phase 3: Execution              | 15%  | Code generation + implementation                    |
| Phase 4: Quality & Iteration    | 5%   | Testing, refinement, divergence prevention          |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ditto190/modme-ui-01](https://github.com/Ditto190/modme-ui-01) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
