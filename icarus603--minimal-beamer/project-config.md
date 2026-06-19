---
trigger: always_on
description: Build a LaTeX Beamer presentation strictly from the bundled minimal Beamer template (assets/template/). Use this skill whenever the user mentions making slides, a presentation, a deck, a talk, or a Beamer/LaTeX presentation — in English ("slides", "deck", "talk", "presentation", "make me a deck about X"), 繁體中文 (「投影片」「簡報」「報告投影片」「做一份簡報」「做投影片」「做 PPT」「做 LaTeX 簡報」「Beamer 簡報」), or 简体中文 (「幻灯片」「演示文稿」「做一份演示」「做演示文稿」「做幻灯片」「做 PPT」「做 LaTeX 演示」「Beamer 演示」) — even if they just dump a topic, a paper, or some no
---


# Minimal Beamer Presentation Skill

You are producing a Beamer slide deck **using the bundled minimal template** at
`assets/template/`. The template is the source of truth: copy it whole and edit
content inside it. Do not invent a different preamble, do not switch themes
unless the user explicitly asks, do not pull in random packages.

This skill is a **workflow**, not a one-shot generator. **You will spend more
turns asking than writing.** That is correct. Every LLM-default choice you
make without asking is a future revision, and revisions cost more than
questions.

## The non-negotiable rules

These exist because every time they were skipped, the user had to redo the
deck:

1. **`AskUserQuestion` is the heartbeat of this skill.** Two places it is
   mandatory and non-skippable:
   - **Before writing any `\begin{frame}`** — clarification is MULTI-ROUND
     (typically 4-5 rounds), not single-round. See Step 1.
   - **After every successful build** — the first turn you send the user
     after `build.sh` succeeds MUST include an `AskUserQuestion` call
     (Step 6). Prose-only handoff that ends with "tell me what you think"
     is a violation.

2. **DO NOT DEFAULT. ASK.** This is the most-violated rule and the cause
   of most revisions. Any decision that affects more than one frame
   (audience, depth, tone, layout style, visual style, narrative arc,
   notation convention, citation policy, mascot author name, *any of
   them*) **MUST be asked, not defaulted**. The only exception is when
   the user says explicitly "你决定" / "use your judgment" / "你看着办" —
   then you can default, AND you must list every default in Step 6.
   "I assumed you'd want X" is the failure mode this rule kills.

3. **NO COLORED BLOCKS, NO AI PLACEHOLDER HEADINGS.** Do not use
   `\begin{block}{标题}`, `\setbeamercolor{block ...}`, custom
   `intuition`/`warningblk` environments, or `tcolorbox`. Do not pepper
   the deck with placeholder section headings like 「关键洞见」「直觉」
   「核心想法」「反直觉的结论」「重要观察」. If a paragraph has a real
   conceptual subtitle (e.g. "两个致命问题", "训练循环") use a bold
   `\textbf{...}` inline lead. If it doesn't, write the paragraph
   directly. Human-written slides do not have colored callout boxes on
   every slide. AI-written slides do.

4. **DEFAULT STRUCTURE = `itemize` AND `enumerate`.** Do not pile prose
   in a slide. If a slide has 3+ ideas, they go in a list. Use ▶ /
   `itemize` for parallel points, `1.`/`enumerate` for ordered steps.
   Prose paragraphs are for transition sentences and final punchlines,
   not for content delivery.

5. **UNIFORM FONT SIZE.** Body text uses `\normalsize` throughout. Do
   NOT sprinkle `\footnotesize` / `\small` / `\scriptsize` to fit content
   on a page. **If a frame overflows, the fix is structural** (split the
   frame, cut bullet points, remove a placeholder heading, move detail
   to a follow-up frame) — not shrinking the font. The only places where
   smaller fonts are legitimate:
   - Code listings (`\scriptsize` or `\tiny` inside `lstlisting`)
   - Reference / footnote-style asides explicitly marked as such
   - The auto-generated `\AtBeginSection` TOC frame (if it exists)
   Even those should be questioned per Step 1.

6. **Never write a single `\begin{frame}` until clarification is done.**
   See Step 1. Slides built from underspecified input look generic and
   wrong, and the user will throw them away.

7. **Never invent facts.** If you do not know the topic well enough to
   defend every claim on every slide, use WebSearch / WebFetch / read
   the source files before writing. Citing a paper you have not actually
   read is forbidden.

8. **The template is the template.** Copy `assets/template/` into the
   working directory and edit `main.tex` in place. Keep the preamble
   structure. Add frames following existing patterns. If something
   requires a new package, ask the user first.

9. **Compile before declaring done.** Run `scripts/build.sh` and resolve
   every error.

10. **Do Step 0 (environment check) first, every time.**

11. **The first PDF is a draft, never a deliverable.** The user — not
    you — decides when the deck is done.

## Script compatibility

All bundled scripts are **bash** (`check_env.sh`, `build.sh`, `preview.sh`,
`clean.sh`). They run directly on **macOS and Linux**. On **Windows**: use
WSL (recommended) or Git Bash; native PowerShell / CMD not supported.

## Step 0 — Check the LaTeX environment (FIRST, every time)

```bash
bash ${CLAUDE_PLUGIN_ROOT}/scripts/check_env.sh
```

Exits with `0` (ready), `1` (partial), or `2` (no LaTeX at all).

- **Exit 0**: proceed.
- **Exit 1**: note what's missing; offer install if CJK needed.
- **Exit 2**: stop. Tell the user what to install:
  - macOS: `brew install --cask mactex` (full) or `basictex` + `sudo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Icarus603/minimal-beamer](https://github.com/Icarus603/minimal-beamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
