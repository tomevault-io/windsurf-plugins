---
trigger: always_on
description: Typeset professional documents and product landing pages: resumes, one-pagers, white papers, letters, portfolios, slide decks, landing pages. Warm parchment, ink-blue accent, serif-led hierarchy. CN uses TsangerJinKai02, EN uses Charter, JA uses YuMincho (best-effort). Triggers on "做 PDF / 排版 / 一页纸 / 白皮书 / 作品集 / 简历 / PPT / slides / Marp / markdown slides / マークダウンのスライド / 落地页 / 官网 / landing page / product page", or "build me a resume / make a one-pager / design a slide deck / turn this into a PDF 
---


# kami · 紙

**紙 · かみ** - the paper your deliverables land on.

Good content deserves good paper. One design language across eight document types: warm parchment canvas, ink-blue accent, serif-led hierarchy, tight editorial rhythm.

Part of `Kaku · Waza · Kami` - Kaku writes code, Waza drills habits, **Kami delivers documents**.

**Update check (non-blocking).** At the start of a task, run `bash scripts/check-update.sh`. It does a read-only version check at most once per day and prints one line when a newer kami is available; relay that line to the user, then continue. It sends no data, and fails silently when offline, sandboxed, or without `curl`. Never let it block the work.

## Step 0 · Load brand profile (if exists)

Check `~/.config/kami/brand.md` (preferred) or `~/.kami/brand.md` (legacy fallback). If found, read `references/brand-profile.md` for the full four-layer application spec (placeholder substitution, session defaults, visual customization, habit notes) and its six guardrails. If no profile exists, continue without interruption.

Key rule: explicit prompt > editorial judgment > habit notes > frontmatter defaults > built-in defaults. Profile fills gaps silently; it never overrides the current conversation.

## Step 0.5 · User project style scan (opt-in)

Run this only when the user explicitly references a sibling project as a visual reference: "like my <project> site", "match the style of <repo>", "use the look from <directory>". Skip silently when no such reference exists.

When triggered, before generating:

1. Locate the referenced project's style files:
   ```bash
   find <referenced-path> -maxdepth 4 \( -name "*.css" -o -name "tailwind.config.*" -o -name "theme.*" -o -name "tokens.*" \) | head -20
   ```
2. Extract: dominant color values (hex / hsl), font stack, spacing scale, border-radius scale. Prefer values declared in CSS variables or design tokens over inline literals.
3. Merge into the in-session brand profile as Layer C (visual customization), not Layer B (session defaults). Do not override an explicit `--brand` flag or values that the user typed in this turn.
4. Report back in one line before continuing: "scanned <project>, extracted N colors / M fonts; using as visual reference."

Skip and fall back to the brand profile defaults if the referenced path does not exist, no CSS-like files are found, or the extraction would conflict with the user's explicit values in the current message.

---

## Step 1 · Decide the language

**Match the user's language.** Chinese -> `*.html` / `slides-weasy.html`. English -> `*-en.html` / `slides-weasy-en.html`. Japanese -> CJK path (`.html` / `slides-weasy.html`) as best-effort, JP Mincho first, visual QA before shipping. Korean -> dedicated `*-ko.html` / `slides-weasy-ko.html` family as best-effort, visual QA before shipping. Reference docs are shared English specs.

When ambiguous (e.g. a one-word command like "resume"), ask a one-liner rather than guess.

| User language | HTML templates | Slides (PDF default) | Slides (PPTX fallback) |
|---|---|---|---|
| Chinese (primary) | `*.html` | `slides-weasy.html` | `slides.py` |
| English | `*-en.html` | `slides-weasy-en.html` | `slides-en.py` |
| Japanese (best-effort) | `*.html` | `slides-weasy.html` | `slides.py` |
| Korean (best-effort) | `*-ko.html` | `slides-weasy-ko.html` | n/a (use `slides-en.py` only if PPTX is required) |
| Other languages (best-effort) | choose CJK or EN path by script coverage, then verify manually | choose `slides-weasy.html` or `slides-weasy-en.html`, then verify manually | use `slides.py` / `slides-en.py` only if PPTX is required |

> Default to the WeasyPrint HTML path; fall back to PPTX (`slides*.py`) only when the user explicitly needs an editable deck.

Always use `CHEATSHEET.md` and `references/*.md` for design, writing, production, and diagram guidance.

Code blocks with `class="language-*"` are highlighted only when optional `Pygments` is installed in the build environment. Without it, PDFs still render and code blocks stay monochrome.

## Step 1.5 · Intent extraction (silent checklist)

Before choosing a template, verify these four dimensions are clear. Do not ask unless 2+ are missing and cannot be inferred from context.

| Dimension | What to extract | Example |
|---|---|---|
| **Purpose** | Why this document exists | Persuade investor vs. align internal team vs. close a candidate |
| **Audience** | Who reads it, what they already know | Technical CTO (skip basics) vs. non-technical board (explain terms) |
| **Constraint** | Hard limits on length, format, tone, or delivery | "One page max", "formal English", "print-ready A4" |
| **Success** | What outcome counts as success | They schedule a meeting / they approve the budget / they understand the architecture |

Rules:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tw93/Kami](https://github.com/tw93/Kami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
