---
trigger: always_on
description: Apply the Diataxis compass to write, restructure, split, classify, review, audit, or migrate technical documentation. Trigger on requests like write docs, organize docs, fix docs, split this page, classify this docs page, audit our docs site, migrate to Diataxis, review this draft, or design a documentation system for an SDK or API.
---


# Diataxis Documentation Skill

Use this skill to turn a documentation request into the right document type, with the right level of detail, for the right reader.

## Core idea

Diataxis separates documentation by two questions — action or cognition, and acquiring or applying skill — and yields four primary forms: tutorial, how-to, reference, explanation. See the compass below for the canonical tool, and [Classification guide](#classification-guide) for the per-form writing details.

## The Diataxis compass

The compass is Diataxis's main classification tool. It reduces a two-dimensional problem to two questions and yields a single answer.

| If the content… | …and serves the user's… | …then it must belong to… |
| --- | --- | --- |
| informs action | acquisition of skill | a tutorial |
| informs action | application of skill | a how-to guide |
| informs cognition | application of skill | reference |
| informs cognition | acquisition of skill | explanation |

To use the compass, ask just two questions:

- Is the content about **action** (practical steps, doing) or **cognition** (theoretical or propositional knowledge, thinking)?
- Is the user **acquiring** skill (study) or **applying** skill (work)?

> "The compass can be applied equally to user situations that need documentation, or to documentation itself that perhaps needs to be moved or improved. Like many good tools, it's surprisingly banal." — [diataxis.fr/compass](https://diataxis.fr/compass/)

The compass is a tool for finding your bearings, not a map of the territory. Diataxis is also published as a 2x2 quadrant diagram on [diataxis.fr](https://diataxis.fr/); this skill calls that the *map*. The map is good for orientation; the compass is good for decisions. The two are not interchangeable.

### Use the compass flexibly

The compass is particularly effective when you think you are doing one thing — or the documentation in front of you seems to be — but feel doubt or difficulty in the work. It forces you to stop and reconsider. Sometimes intuition provides an immediate answer that is also wrong.

Do not get fixated on the exact names. If a question feels ambiguous, both readings may be valid. Apply the compass at any scale: at the level of a single sentence, a section, an entire page, or a whole documentation set.

The questions can be used in different ways:

- "Do I think I am writing for *x* or *y*?"
- "Is this writing in front of me engaged in *x* or *y*?"
- "Does the user need *x* or *y*?"
- "Do I want to *x* or *y*?"

### Apply the compass to existing documentation

The compass is just as useful for auditing existing pages as for greenfield work. For each page, ask the two questions. If the page's current form does not match the answer, the page needs to be moved or rewritten — not labelled differently.

## Quick decision tree

The compass above is the canonical tool. The tree below is a quick aid for the most common cases.

```text
What is the reader trying to do right now?
│
├── Acquiring a new skill from scratch?
│   ├── With a complete, guided lesson → Tutorial
│   └── With the shortest path to first success → Quickstart
│
├── Applying a known skill to a real task?
│   ├── General task with a clear goal → How-to
│   └── Stuck on an error → Troubleshooting
│
├── Looking up an exact fact, field, command, or limit? → Reference
│
└── Trying to understand why or how it works?            → Explanation
```

If the request hits more than one branch, split it into multiple documents rather than blending them. The same questions can be applied to existing pages: when a page's content disagrees with its current form, the page needs to be moved, not relabelled.

## When to use this skill

Use this skill when the task is to:

- write, rewrite, restructure, or review a documentation page
- apply the Diataxis compass to classify a page or a request
- split a single messy page into the right Diataxis forms
- audit or migrate an existing documentation set, one page at a time
- design a full documentation system for an SDK, API, CLI, or product (using the compass as a guide, not as a plan)
- map user questions to the correct documentation form

## When NOT to use this skill

Do not use this skill for:

- marketing copy, blog posts, or non-technical writing
- purely visual content such as UI mockups, slides, or design specs
- code review, refactoring, or implementation work
- writing a single paragraph or a short response that does not need a document structure
- translations of finished documentation that already follows a clear structure

## Guiding principles

- Clarity: write in simple, direct language.
- Accuracy: keep facts, code snippets, and document details up to date.
- User centricity: optimize for the reader's goal, not the author's internal structure.
- Consistency: keep tone, terminology, and formatting aligned across the document set.

## Workflow

For a single page, work in this order:

1. Identify the reader.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [88lin/diataxis-docs-skill](https://github.com/88lin/diataxis-docs-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
