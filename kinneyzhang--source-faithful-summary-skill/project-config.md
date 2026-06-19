---
trigger: always_on
description: Use when summarizing text, video transcripts, audio transcripts, podcasts, lectures, interviews, demos, or meetings where the output must preserve the source's main thread, attention weight, decision model, concrete examples, and evidence instead of becoming a polished but source-drifting essay.
---


# Source-Faithful Summary Skill

## Overview

Use this skill for source-grounded summaries of text, video transcripts, audio transcripts, podcasts, talks, interviews, demos, workshops, lectures, meetings, and other transcript-like source material.

The goal is not to write the most elegant essay. The goal is to preserve what the source actually does: the speaker's main thread, attention weight, concrete examples, sequence, failure modes, actions, and decision model.

A summary can be factually plausible and beautifully written while still being wrong for the task if it replaces the speaker's actual flow with the summarizer's cleaner framework.

Core rule:

```text
Faithfulness to the source > elegance of argument
Narrative weight > concept density
Concrete action/problem chain > abstract framework
Timestamps / section refs / evidence > vibes
Speaker/author/participant decision model > isolated tips
```

## What This Skill Optimizes

A good source-grounded summary is not merely compressed source text. It should recreate the understanding a careful reader, viewer, or listener would get after engaging with the source.

Optimize for five kinds of fidelity:

1. **Main-thread fidelity** — what the source is actually driving toward.
2. **Attention fidelity** — what the source spends time on, repeats, or uses to explain later actions/claims.
3. **Causal fidelity** — why one observation leads to the next decision, claim, or forecast.
4. **Action/argument fidelity** — what the speaker/author/participants actually do or argue, in source order.
5. **Decision-model fidelity** — the constraints, tradeoffs, failure modes, and uncertainty behind the source's recommendations or judgments.

For workflow/practice sources, named concepts must not be treated as decorative keywords. A concept is important if it controls later decisions.

Example:

```text
smart zone / dumb zone
```

In an AI coding workflow talk, this is not just a definition about context degradation. It may explain why tasks are sized small, why context is cleared, why external artifacts like PRDs/Kanban boards matter, and why review should happen in a fresh context.

Default extraction primitive:

```text
failure mode → underlying constraint → speaker's judgment → concrete action → tradeoff/limit
```

## When to Use

Use when the user asks to:

- summarize a YouTube, Bilibili, Vimeo, podcast, interview, lecture, demo, workshop, or recorded meeting;
- rewrite or publish a source-based summary;
- audit whether an existing summary missed the original material's key points;
- extract a speaker's workflow, examples, failure modes, or practical advice;
- turn a long transcript into a public article or private note;
- compare a summary against a source transcript;
- preserve what the source says before writing independent commentary.

Also use when the source is not literally video but has a strong original sequence, such as an interview article, lecture notes, meeting transcript, or conference talk transcript.

Do **not** use this as the primary workflow when the user explicitly wants:

- a free-form opinion essay inspired by a source;
- a broad research article that uses the material as one source among many;
- creative rewriting that intentionally departs from the source;
- marketing copy where source fidelity is not required.

If commentary is needed, first produce the source-faithful summary, then add a clearly separated section such as “Analysis”, “Implications”, or “What this means for us”.

## Required Inputs

Prefer one or more of:

- transcript with timestamps;
- source URL plus transcript tool access;
- existing summary to audit;
- target audience and output length;
- whether the result is public-facing or private notes;
- desired language.

If the transcript is unavailable, state the limitation explicitly and summarize only from available material.


## Output Language Policy

Source language and output language are separate decisions. Do not assume the output language should match the source language.

Use this priority order:

1. **Explicit user request wins.** If the user says “write in Chinese”, “English output”, “用中文”, “生成英文版”, or names a target locale, use that language.
2. **Conversation language next.** If the user does not specify, write in the language the user used to ask for the summary. A Chinese request gets Chinese output; an English request gets English output.
3. **Existing artifact language next.** If revising an existing article or note, preserve its language unless the user asks to change it.
4. **Source language last.** Only mirror the source language when none of the above is available.

For multilingual or international users, ask only when the language signal is genuinely ambiguous. Otherwise choose and state the choice in metadata, for example:

```markdown
Output language: Chinese, chosen from the user request language.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kinneyzhang/source-faithful-summary-skill](https://github.com/Kinneyzhang/source-faithful-summary-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
