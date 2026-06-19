---
trigger: always_on
description: Use this skill when the user wants to strip AI-writing tells from prose — triggers include "ungptize", "remove AI tells", "make this less ChatGPT-y", "de-AI this", "this sounds like AI / GPT / an LLM", "rewrite to sound human", "scrub the GPT-isms", "less corporate", "too polished", or simply pasting prose and asking for a more human voice. Detects and rewrites the patterns Wikipedia catalogues as signs of AI writing — overused vocabulary (delve, pivotal, tapestry, robust, vibrant, etc.), avoida
---


# ungptize

Take prose the user suspects is AI-shaped and rewrite it to remove the linguistic tells while preserving meaning, voice, and register.

Source: the *Language and grammar* and *Style* sections of Wikipedia's [*Signs of AI writing*](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), plus the cadence tells that LinkedIn-style AI prose is built on (which Wikipedia doesn't cover but every model trained on the open web reproduces).

## How this skill works: two phases

**Phase 1 — Inventory (find everything).** A detector subagent reads the prose and produces an exhaustive inventory of every candidate smell. The subagent has no rewrite tools; its only job is enumeration.

**Phase 2 — Rewrite (act on the inventory).** The main agent rewrites only the rows the inventory flagged `rewrite`. The change log is the inventory, filtered.

Why two phases, and why a subagent for phase 1: when detection and rewriting happen in the same context, the agent skims, fixes obvious tells, and stops. Splitting detection into a separate subagent forces enumeration before action — and because the subagent literally can't rewrite, the phase boundary is enforced by tool access, not just discipline.

## Why the agent is the detector, not regex

Earlier versions of this skill tried to detect structural tells (negative parallelism, antithesis, copulative avoidance) with regex. That was the wrong layer. Each pattern has too many surface forms — "not X, but Y" / "isn't X — it's Y" / "wasn't X; it was Y" / "X, not Y" / "no X, no Y, just Z" — and the verdict ("decorative or genuine contrast?") is semantic, not lexical.

The right detector is a language model reading the prose. The recognition happens by *seeing* the shape, not by matching a regex. Even mechanical-looking signals (vocabulary density, em-dash count, curly-quote use, one-sentence-paragraph rhythm) are simple enough to count by reading — adding a script for them costs more than it saves, and the temptation to "trust the count" hides cases where the same number means something different in context.

## Workflow

- [ ] **1. Read the input.** Identify the language. Note voice, register, cadence, and the kind of document (essay, post, README, RFC, FAQ) — register decides which patterns are tells vs. legitimate.
- [ ] **2. Phase 1: spawn a detector subagent.** Use the `general-purpose` subagent. Give it: the prose (line-numbered), the pattern catalog from this file, and the inventory format. Instruct it to produce the full inventory and return only that — no rewrite, no commentary. See "Detector subagent prompt" below.
- [ ] **3. Apply the clustering rule** to the returned inventory. Single neutral hits get `keep`. Decorative or clustered hits get `rewrite`. Genuine semantic contrasts get `keep`.
- [ ] **4. Phase 2: rewrite.** Act only on `rewrite` rows. Preserve meaning, voice, register. Prefer the simplest copulative ("is"/"has"). Don't invent content to fill gaps.
- [ ] **5. Verify** that meaning is preserved and the rewrite reads in the user's voice (or in a neutral human voice if the user is anonymous).
- [ ] **6. Output** the inventory, then the rewritten prose, then a brief change log (inventory filtered to `rewrite` rows).

## Inventory format

One row per finding. Fixed shape, scannable:

```
L<line> · <category> · "<exact phrase>" · <verdict> · <one-line why>
```

- `<category>` ∈ `vocab`, `copulative`, `neg-parallel`, `tricolon`, `elegant-var`, `concession-block`, `cadence`, `em-dash`, `curly-quote`
- `<verdict>` ∈ `rewrite`, `keep`, `borderline`
- `<one-line why>` — terse justification, no hedging

Example rows:

```
L1  · vocab        · "pivotal"                                              · rewrite   · decorative, no work being done
L7  · neg-parallel · "isn't that the AI is 'smart' — it's that maintainer…" · rewrite   · decorative antithesis, no real contrast
L9  · neg-parallel · "the fix is a rule, not a patch"                       · rewrite   · `X, not Y` decorative form
L12 · tricolon     · "better, faster, and more honest"                      · rewrite   · adj triple, "faster" carries no weight
L14 · vocab        · "key insight"                                          · keep      · single neutral usage, doing real work
L18 · em-dash      · 10 dashes across 850 words                             · rewrite   · flooding, ~1 per 85 words
P1-P8 · cadence    · 6 of 8 paragraphs are one sentence                     · rewrite   · LinkedIn AI cadence
```

`L<n>` for line-anchored findings; `P<n>` for paragraph-level (cadence, structural). Walk the prose end-to-end before writing the inventory. **If the inventory is incomplete, the rewrite will be incomplete.**

## Detector subagent prompt

When spawning the phase-1 subagent (`general-purpose`), use a prompt of this shape:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnslavik/ungptize](https://github.com/johnslavik/ungptize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
