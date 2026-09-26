---
trigger: always_on
description: How we write docs in this repo. The goal is **agent-readable, human-skimmable references** — not marketing, not aspirational text, not a notebook for in-flight work.
---

# Documentation Conventions

How we write docs in this repo. The goal is **agent-readable, human-skimmable references** — not marketing, not aspirational text, not a notebook for in-flight work.

This file is the meta-doc. If you change docs structure, naming, or voice, change it here first, then update the affected docs.

---

## Audience

Every doc in `docs/` targets **two readers, in this priority order**:

1. **Coding agents** (Claude, Codex, etc.) trying to make a correct change in this codebase.
2. **Humans** (the author and contributors) skimming for orientation or details.

Agents read top-to-bottom and rely on **concrete file paths, code shapes, and invariants**. Humans skim for structure. Both readers benefit from the same thing: short sections with clear names, real examples, and no waffle.

If a sentence does not help one of those two readers make a decision or correct a misunderstanding, **delete it**.

---

## Three doc types

Every file under `docs/` is one of these. The folder it lives in says which type it is.

### 1. Top-level (`docs/*.md`)

System-wide references read first. The cornerstone set:

- `architecture.md` — what the system is, how the layers fit
- `design.md` — the visual system (tokens, components, principles)
- `server.md` — server-side deep dive
- `editor.md` — admin + canvas editor deep dive
- `CONVENTIONS.md` — this file (docs conventions)

Top-level docs are **long-lived and authoritative**. They describe the system as it currently is. They never describe in-flight work, future plans, or alternatives that were considered. New ones are rare — propose before adding.

### 2. Features (`docs/features/*.md`)

One doc per first-class feature. "Feature" means a coherent capability with a name a user would recognize: plugin system, visual editor, publisher, media, visual components, auth, etc.

Feature docs explain **what the feature is, how it's built, where its code lives, and how to extend it**. They are not specs of what to build — they are descriptions of what exists.

### 3. Reference (`docs/reference/*.md`)

Short, focused, agent-targeted cookbook pages for primitives and patterns that get reused across features: the `NodeTree` primitive, TypeBox patterns, UI primitive usage, design tokens, database dialect rules, the architecture gate tests.

A reference doc answers one question: "How do I correctly use / implement X?"

---

## Folder layout

```
docs/
├── README.md                   Index — where to start, what to read
├── CONVENTIONS.md              This file
├── architecture.md             System overview
├── design.md                   Visual design system
├── server.md                   Server deep-dive
├── editor.md                   Admin + canvas editor deep-dive
│
├── features/                   "What X is and how it works"
│   ├── plugin-system.md
│   ├── publisher.md
│   ├── visual-components.md
│   ├── content-storage.md
│   ├── media.md
│   └── ...
│
├── reference/                  Short cookbook pages
│   ├── page-tree.md
│   ├── ui-primitives.md
│   ├── design-tokens.md
│   ├── typebox-patterns.md
│   ├── database-dialects.md
│   ├── architecture-tests.md
│   └── ...
│
├── deployment/                 Operator docs (platform targets + generic hosts)
├── e2e/                        Agent-run browser test protocols (kept as-is)
└── plans/                      In-flight design plans (transient)
```

**Plans are not docs.** They live in `plans/` because they describe work-in-progress decisions. When a plan ships, the resulting state goes into `features/` or `reference/` — the plan itself is deleted or archived. Never read `plans/` to learn how the system works.

---

## Required shape

Every doc — top-level, feature, or reference — follows this skeleton:

```md
# <Title>

<One-sentence statement of what this doc covers.>

<One paragraph: the problem it solves / the system it describes, in the form
"X is Y that does Z." No history. No "we used to ..." No marketing.>

---

## TL;DR

<Three to ten bullets or a small table. The reader gets the answer here. The
rest of the doc justifies and extends it.>

## <Body sections>

<Specific, named sections. See "Section choices" below.>

## Related

- `docs/<other>.md` — when to read instead / next
- Source-of-truth files: `path/to/file.ts`
- Gate tests: `src/__tests__/architecture/<file>.test.ts`
```

The `Related` section is mandatory. It tells the reader what to read next and where the source of truth lives.

### Section choices by doc type

**Top-level docs** typically have:
- TL;DR
- Layout / Architecture
- Layer responsibilities (table)
- Data flow (ASCII diagram)
- Invariants and gates
- Where things live (file map)
- Related

**Feature docs** typically have:
- TL;DR
- Architecture (what lives where, what depends on what)
- Data flow / lifecycle
- Adding a new X (cookbook)
- Forbidden patterns / gotchas
- Related

**Reference docs** typically have:
- TL;DR
- The shape (type signatures, file paths, the canonical example)
- How to use it (one or two cookbook examples)
- Forbidden patterns
- Related

---

## Voice and content rules

### Hard rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoreBunch/Instatic](https://github.com/CoreBunch/Instatic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
