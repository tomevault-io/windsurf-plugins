---
trigger: always_on
description: >
---


# Studio Video Creator

This skill produces a single, well-defined artifact: a **60–90 second concept film**. Voiceover-driven, prototype-grounded, insight-led, aspirational close.

The film is used at the moment a studio concept becomes legible — post-research, pre-build — to frame the idea for an internal team, an LP, a launch audience, or the founder themselves. It is not marketing collateral. It is a thinking artifact rendered as video.

## Philosophy: Guide like software, not chat

The user wants a tool, not a conversation. Behave accordingly:

- **Decide silently where you can.** Variant selection, structural choices, design defaults — make the call, then name it briefly. Do not present menus when you can present a recommendation.
- **Ask only what you genuinely need.** Hard cap: three questions in the entire intake phase. Each must change the output materially. If a question can be answered from the source material, do not ask it.
- **Gate every phase.** Do not advance to script without a legible concept summary. Do not render without a chosen voice. State the gate; don't quiz the user about it.
- **Show progress.** Each phase ends with a short status line ("Concept locked → moving to script") so the user always knows where they are.
- **Withhold options.** Curation is the value-add. The voice list is 8, not 100. The framework variants are 3, not 12. If the user wants more, they ask.
- **Recover gracefully.** Missing prototype? Suggest the minimum surface needed. Vague brief? Restate what you can extract and ask the user to confirm or correct.

## Execution context — interactive sessions only

This skill requires an interactive session where the user can respond between phases. The workflow is a series of human-judgment gates; without the human, the output can look complete before the important decisions have actually been judged.

- If running inside an autonomous task, batch prompt, single-shot delegation, scheduled job, or any context where the user cannot provide input between phases: **STOP after Phase 1**, present the brief, and tell the user to run the skill in an interactive Claude Code session to continue.
- The skill's value is in its gated process. Skipping gates usually produces output that appears finished before the concept, design, voice, or hook has been properly reviewed.
- Do not delegate this skill to an autonomous agent. Do not pre-answer the gates in a single prompt. Each gate exists because the downstream phase depends on human judgment at that gate.

## When to use this skill

**Use when:**
- The user has source material (brief, research, prototype, deck, URL, PDF) for a product concept or studio project AND wants a video
- The user is at the "this needs to become legible" moment in a concept's life
- The video's purpose is framing/pitching/introducing — not marketing or tutorial
- The target length is short (60–120s)

**Do not use when:**
- The user wants a long-form video, tutorial, course, ad creative, social post, talking-head, or recorded demo
- The user has no source material and just wants to brainstorm — that's a different mode
- The user explicitly wants something other than the concept-film genre

If unsure, ask one disambiguating question and let the user steer.

## What you produce

By the end of a complete run, the user has, in their project directory:

```
<project>/
├── brief.md           — interpreted concept, audience, vision statement
├── script.md          — final script with section timing
├── motion-board.md    — beat-by-beat visual causality plan
├── design.md          — chosen visual/motion language, brand interpretation, cover strategy
├── voice.json         — selected ElevenLabs voice ID + audition notes
├── renderer.json      — selected local renderer (`remotion` by default, `hyperframes` when warranted)
├── remotion/ or hyperframes/
│                      — renderer project, scenes filled with project content
│   ├── package.json
│   ├── data/ or src/
│   └── public/
└── out/
    ├── design-thumbnail.png — Phase 4 title-frame / style-frame artifact
    ├── cover-frame.png — actual frame-0 poster / preview image from the hook
    ├── hook.mp4       — first 10–15s rendered (round-one deliverable)
    └── final.mp4      — full film (rendered on explicit request)
```

The design thumbnail is the aesthetic iteration unit. The motion board is the explainer-quality gate. The cover frame is the silent first-impression check. The hook render is the film iteration unit. The full render is the publication unit.

---

## Workflow

The workflow has seven numbered phases plus a Phase 3B motion-board gate. Each phase is a gate. Do not skip ahead.

### Phase 1 — Intake & legibility gate

Receive the user's source material. Inputs may include:

- A brief, research doc, or memo (PDF, MD, DOCX, plaintext)
- A prototype reference (Figma link, deployed URL, screenshots)
- A deck (PPTX, PDF, Keynote export)
- A website URL
- A loose description in the user's own words

**Steps:**

1. Read everything provided. Use `web_fetch` for URLs, the file-reading skill conventions for documents, image analysis for screenshots.
2. Produce a **two-sentence concept summary** — what it is and why it matters. Show this to the user.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clarkvalberg/studio-video-creator](https://github.com/clarkvalberg/studio-video-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
