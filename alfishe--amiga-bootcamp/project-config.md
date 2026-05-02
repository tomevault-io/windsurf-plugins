---
trigger: always_on
description: > **Audience**: AI coding assistants and human contributors.
---

# AGENTS.md — Technical Article Quality Standards

> **Audience**: AI coding assistants and human contributors.
> Read this before writing or expanding any `.md` file in this repository.

---

## Language & Spelling

- **American English only** — `color`, `behavior`, `initialize`, never `colour`, `behaviour`, `initialise`
- Use clear, direct technical prose — no filler, no hedging, no marketing language
- Prefer active voice: "The Blitter copies data" not "Data is copied by the Blitter"
- **Hexadecimal**: Use Motorola syntax (`$DFF180`) for hardware registers and memory addresses, not C-style (`0xDFF180`), unless explicitly within a C code block.

---

## Pre-Flight: Knowledge Base Scan (MANDATORY)

Before writing or expanding any article, you **must**:

1. **Read the root [README.md](README.md)** — it contains the full Documentation Map with every article in the repository. Understand what already exists to avoid duplicating content and to identify cross-linking opportunities.
2. **Scan the section's `README.md`** — know what sibling articles exist in the same folder. Your article should complement, not repeat, adjacent content.
3. **Search for related content** — use grep or file listing to find existing mentions of your topic across the repository. If another article already covers a subtopic in depth, link to it rather than rewriting it.
4. **Check for established patterns** — look at 2–3 exemplary articles in the repository (see "What Makes an Exemplary Article" below) to match style, depth, and structure.

> [!IMPORTANT]
> Every article must exist within the knowledge graph. Orphaned articles are unacceptable. Update the root README's Documentation Map when adding new articles.

---

## Article Structure

Every article must follow this skeleton. Omit sections that genuinely don't apply, but the **Overview** and **Navigation** are mandatory.

### 1. Navigation Breadcrumb (line 1)

```markdown
[← Home](../README.md) · [Section Name](README.md)
```

### 2. Title

Use a single `#` heading. Include the subject and a subtitle with key subtopics:

```markdown
# Memory Management — AllocMem, FreeMem, MemHeader
# Copper Programming — Deep Dive
# Animation — GEL System: BOBs, VSprites, AnimObs
```

### 3. Overview

The first section after the title. Two distinct styles depending on article type — choose before you write.

---

#### Type A: Concept Articles (history, chipset, OS subsystem, architectural deep-dives)

These articles answer *why* something exists and *what changed because of it*. The overview should be **scannable and alive** — a reader understands scope in under 10 seconds *and wants to keep reading*.

**Planning checklist** (answer these, then write prose that embeds the answers naturally):

| Question | What it means |
|---|---|
| **What is it?** | Define the thing, its scope, and where it lives. |
| **What was the problem?** | Why does this exist? Concrete pain, not abstract need. |
| **How does it work / how did it evolve?** | Core mechanism or historical arc. |
| **Why does it matter?** | What changed? Numbers, not adjectives. |

**Voice:**

- **Open with a hook.** A contradiction, a surprise, a concrete image. "The Amiga had the best graphics chipset in the world — and a CPU that couldn't do math." Not "Floating-point arithmetic is the computational foundation of..."
- **Write like you're explaining to a smart colleague at a bar.** Short sentences. Concrete nouns (chips, scanlines, frames, hours). No filler.
- **Bold vivid details, not section labels.** Bold `**two and a half video scanlines**`, not `**The problem.**`.
- **Paragraphs tell a story.** Hook → problem → solution arc → punchline. 2–5 sentences each.
- **Every paragraph gets a blank line after it.** Never a monolithic block. If you have 6 sentences about four CPU generations, split them into 2–3 paragraphs — one per phase or per idea. The blank lines are not optional; they are how the reader breathes.

Bad (form-filling):
> **What it is.** Floating-point arithmetic is the computational foundation of 3D rendering...
> **The problem.** The 68000 CPU was integer-only...
> **The solution.** First, Motorola shipped the MC68881...

Good (alive prose — FPU article):
> In 1985, the Amiga had the best graphics chipset in the world — and a CPU that couldn't do math. The 68000 was an integer-only processor: brilliant at moving pixels, helpless at trigonometry. Ask it to compute `sin(0.7)` and it would freeze for **two and a half video scanlines**...
>
> That was the problem the FPU solved. Motorola shipped four generations between 1984 and 1994, each designed under the same brutal constraint: floating-point circuits are **30× more complex** than integer circuits...

---

#### Type B: Reference Articles (register catalogs, file format specs, LVO tables, API listings, error code tables)

These articles exist to be **looked up**, not read cover-to-cover. The overview must be minimal: state what the thing is, where it lives, and get out of the way.

**Template (one paragraph, 2–4 sentences):**

> [Thing] is [definition — one sentence]. It lives in [library / chip / NDK header path / file]. See [cross-link] for the broader context.

**Rules:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alfishe/amiga-bootcamp](https://github.com/alfishe/amiga-bootcamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
