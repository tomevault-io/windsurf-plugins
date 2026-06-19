---
trigger: always_on
description: Use when a user wants to turn lecture slides, exercises, and past papers into interactive HTML study notes for exam prep. Triggers on phrases like "速通笔记", "交互式 html 笔记", "把课件做成网页", "把我当三岁小孩讲", "exam crash course", "interactive study notes", "lecture-to-html", "study guide from PDFs", or any combination of (a course folder / lecture materials) + (review / exam / cram / notes intent). Also fires on improvement requests for existing interactive notes ("improve these notes", "expand the cheatsheet",
---


# Interactive Study Notes

This skill converts a folder of lecture materials (PDFs, slides, past papers, homework) into a set of polished, interactive HTML study notes — the kind you'd actually want to cram from the night before an exam.

The output is **offline-capable plain HTML/CSS/JS** — no build step, no framework, double-clickable, OneDrive-friendly. Math renders via KaTeX. Interactive components include flip-card term drills, MCQ self-quizzes with instant feedback, step-by-step algorithm animations, sidebar navigation with scroll progress, and per-lecture cheat sheets.

## When to trigger

Pattern-match these (mix freely across languages):

- "速通笔记" / "复习笔记" / "考前笔记" / "exam crash course" / "study guide" / "cheat sheet"
- "期末降至 / 死到临头" + any course materials
- "把 [PDF/PPT/课件/lectures] 做成 html / 网页 / 交互式 / interactive"
- "把我当三岁小孩教" / "ELI5" / "深入浅出" / "explain like I'm five"
- Any mention of converting lecture slides into web-viewable study materials
- Improvement requests on existing interactive notes ("看一下当前工作区的交互笔记，然后予以改进", "the notes are too sparse", "add more practice questions")

When triggered, **read this whole file plus `references/workflow.md`**, then proceed. The skill is a workflow, not a one-shot generator.

## The product shape (non-negotiable)

These constraints come from real student feedback and have been validated across multiple courses:

1. **Per-lecture HTML files + one `index.html` hub** in an `interactive note/` (or similarly named) folder. One file per chapter, not a monolithic single-page app.
2. **Offline-capable**: works by double-clicking the file (`file://` protocol). No build step, no npm, no CDN dependencies for *core* functionality. CDN is allowed only for KaTeX/MathJax (math rendering) — the page must still display readable text if offline.
3. **Shared CSS/JS** when producing ≥3 lectures: copy this skill's `assets/notes-<skin>.css` + `assets/notes-<skin>.js` into `interactive note/assets/` and reference via relative path. Don't reinvent the components — copy and adapt.
4. **Pedagogical model**: ELI5-first (everyday analogy before formal definition) + exam-focused (every concept ends with self-quiz) + **zero prior knowledge** (write for a reader who has NEVER opened the lecture slides — not for someone refreshing what they already saw. Every algorithm name, abbreviation, metric, data structure, and math symbol must be expanded inline on first appearance. No undefined-term chains. Past complaint: "假设我看过课件了，但是我希望是把我当作没看过课件的人来教"). Every concept card follows a fixed 6-section structure (see below).
5. **HOMEWORK + PAST PAPERS are MANDATORY inputs**. If the workspace has a `hw/`, `homework/`, `assignment/`, `past paper/`, `previous exam/`, or similar folder, you MUST read every file and mine for: (a) recurring question types, (b) worked examples to drop into 🌰, (c) traps to drop into ⚠️, (d) MCQs/short-answer items to seed 🎯 final quiz. Failing to do this produces "too abstract" notes — the textbook explains *concepts*, the homework reveals *what gets tested*.
6. **"Dark-horse questions" (🌑 暗箭考点)**: After mapping past-paper coverage, explicitly identify slide content that appears in the lecture but is **absent from past papers/homework/exercises**. These untested-but-in-scope topics are the **most likely surprise questions** — they haven't been "burned through" yet. The 🎯 final quiz of each lecture MUST include 2-3 questions on these dark-horse topics, not only past-paper paraphrases.

## Canonical concept card (6 sections)

Every concept card in a lecture page follows this scaffold. The emojis are part of the brand — keep them as section markers (they make scanning the page much faster):

1. **🍼 ELI5 / 大白话** — Plain-language analogy first. No jargon. Like explaining to a curious 5-year-old.
2. **📖 Formal definition / 正式定义** — Formal definition with technical terms preserved in their original language (e.g., keep "Apriori", "support", "lift ratio" in English even when surrounding prose is in another language — exams usually use the canonical English term).
3. **🌰 Example / 例子** — A concrete worked example with real numbers, or a famous case study. For math/algorithm courses: actual numerical walk-through. For concept courses: a famous case (e.g., Kodak/Blockbuster, Netflix, etc.).
4. **⚠️ Exam trap / 考点陷阱** — The subtle distinction the student is likely to get wrong. Sourced from past papers when available; from common misconceptions otherwise.
5. **🔄 Flip-cards / 翻卡** — 2-4 key terms as 3D flip-cards (click to flip). Front = term, back = definition. Drills vocabulary.
6. **❓ Self-quiz / 自测** — 1-3 MCQs with **instant feedback**: click an option → immediately get green/red + reveal explanation. No "submit at end" button.

At the END of each lecture page, two more sections:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SHKinsem/Interactive-Study-Notes](https://github.com/SHKinsem/Interactive-Study-Notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
