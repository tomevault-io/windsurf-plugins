---
trigger: always_on
description: Generate polished, standalone HTML study notes AND full step-by-step homework solutions for academic subjects, especially STEM (physics, math, chemistry, engineering, CS). Trigger on requests to create study notes, learning notes (学习笔记 / 复习笔记), a study guide, lecture notes, or self-study reference — e.g. "make me notes on X", "help me learn X", "summarize X for an exam"; when the user uploads a PDF textbook or course material and asks for review notes; or when the user gives homework / 习题 (作业题) 
---


# Study Notes Skill

Generates detailed, exam-ready HTML study notes — and full homework solutions — that are visually polished, mathematically rigorous, and pedagogically structured, modeled after the best university-level reference materials.

**Default audience assumption**: treat the reader as someone encountering this subject for the first time. Write as if they attended lectures but are confused and need everything explained clearly — intuition first, then rigor, then worked examples. Never assume prior knowledge beyond what is stated as a prerequisite.

**Path convention (read this first).** The examples below use `/mnt/user-data/uploads/` (inputs) and `/mnt/user-data/outputs/` (outputs) as **placeholders** — these are the Claude.ai web paths. In **Claude Code** (or any other environment) substitute the real paths: read inputs from wherever the user's files actually are, and write the final HTML to the current working directory (or a directory the user names). Every helper script in `scripts/` takes paths as arguments, so nothing is hard-coded — call them with the paths that exist in your environment.

**Environment portability.** Commands below say `python3` and end with `present_files` — both are Claude.ai-web conventions. On a local machine (e.g. Windows) the interpreter is usually just `python`, and there is no `present_files` — instead, simply report the absolute path of the finished HTML so the user can open it. Use whichever interpreter name resolves in the current environment.

---

## 0. Workflow Routing — 先判断模式（START HERE）

This skill is a **multi-branch workflow**. Before generating anything, read the input, then route to exactly one mode.

> On the relationship to Claude Code "dynamic workflows": a SKILL.md cannot *be* a dynamic workflow (those are JavaScript-orchestrated subagent runs). What this routing does is make the skill *behave* like one — a decision tree over input types. **All three modes should additionally run their generation as a workflow** — plan → fan-out → **verify** → assemble — rather than one giant pass. This is what fixes shallow notes and wrong answers. See **`references/workflow-orchestration.md`** (mandatory for MODE A and MODE B) and the scale tip in MODE C.

| Input | User's intent | → Mode |
|---|---|---|
| 课本 / 讲义 PDF (text or scanned images) | "出学习笔记 / 复习笔记 / review notes" | **MODE A** — PDF → 学习笔记 |
| 几道作业 / 习题 | "用这些题帮我重新学这一章" | **MODE B** — 作业题 → 章节学习笔记 |
| 一份作业 / 习题 | "给我每道题的解答 / 答案" | **MODE C** — 作业题 → 全部题目解答 |
| 只有一个主题，无文件 | "帮我做 X 的笔记" | **MODE A (scratch)** — 从零生成 |

**Routing rules**

1. **Read inputs first.** Extract PDF text/images (Step 0) and read any problem images before deciding.
2. **Detect intent from the verb, not just the attachment**: 学 / 复习 / 笔记 / 整理 → notes (A or B); 解 / 答案 / 做题 / solution → solutions (C).
3. **If ambiguous** (problems given, no clear verb), ask **exactly one** question:
   *"要我 ① 用这些题帮你重新学这一章（出学习笔记），还是 ② 直接给出每道题的详细解答？"* — then route.
4. **Combined inputs:**
   - PDF **+** 作业 → run MODE A on the PDF, then append a MODE C "习题详解" section to the same HTML.
   - "学这一章，顺便把作业做了" → MODE B notes, with each homework problem embedded as a **collapsible worked-example card** inside the section that teaches its concept.

All modes share the same design system, math-rendering rules, post-generation checks, and file-generation strategy described later in this file. The modes only differ in **what content to produce and how to structure it**.

---

## 0.5 Run generation as a workflow (MANDATORY for MODE A & MODE B)

Do not generate a whole document in one linear pass. Run each mode in four phases — this is the
fix for "笔记空泛" and especially "题做错了". Full details in **`references/workflow-orchestration.md`**; the essentials:

1. **Plan (one pass):** read the source, build the TOC + concept list, and write a **shared spec**
   — notation table (symbols, units, sign conventions from the source), section color plan, and
   the design/HTML/math rules. Every later unit receives this spec so parallel output stays consistent.
2. **Fan-out:** generate **one section per concept** as its own unit (dynamic workflow if the word
   "workflow" is in the run, else parallel subagents via the Task tool, else sequential for small
   topics). Focused units go deeper than one overloaded pass.
3. **Verify (the decisive step):** check each fragment with `scripts/build_and_check.py`, and
   **verify every answer** — every worked example (MODE A) and every homework solution (MODE B) —
   with the **per-problem verification checklist** in `references/workflow-orchestration.md`.
4. **Assemble + coherence pass:** concatenate in order, then one pass for consistent notation,
   no duplicate/clashing sections, working TOC links; re-run the post-generation checks.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eric6286/study-notes](https://github.com/Eric6286/study-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
