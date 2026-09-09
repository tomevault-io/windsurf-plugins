---
trigger: always_on
description: This directory holds the official Pest documentation. When writing or editing any file here, match the established voice of the docs exactly. Consistency of tone matters as much as technical accuracy — a page that is correct but off-voice still needs rework.
---

# CLAUDE.md

This directory holds the official Pest documentation. When writing or editing any file here, match the established voice of the docs exactly. Consistency of tone matters as much as technical accuracy — a page that is correct but off-voice still needs rework.

The sections below describe that voice in detail. They are derived from how the docs already read; treat them as house style, not suggestions.

The house style deliberately mirrors the [Laravel documentation](https://github.com/laravel/docs) — the same permission-granting modality ("you may"), the same colon-into-code rhythm, the same warm-expert register, and the same structural conventions. When in doubt about a construction, ask: would a `laravel/docs` page phrase it this way?

## The Voice in One Sentence

Write as a **warm, confident expert personally guiding a capable peer** — conversational but precise, encouraging without being cutesy, and relentlessly consistent in sentence scaffolding. The reader is always the protagonist; Pest is the helpful tool doing work on their behalf.

The dominant rhetorical move is: **state the reader's likely need → name the tool → show how → explain what just happened.**

## Point of View

- **Second person ("you" / "your") is the default.** The reader is the actor in nearly every instructional sentence: "you may use", "you should define", "your test suite", "your project", "your expectations". The possessive "your" personalizes ownership relentlessly.
- **Use "we" / "our" / "let's" for shared walkthroughs.** When walking through an example, reason alongside the reader: "Let's write our first test", "let's imagine your project features a `sum` function", "we'll cover each of these in detail". This mode clusters in introductions and worked examples; the "you" mode dominates instructions and options. The two coexist within a section — set up a scenario with "let's imagine…", then hand control back with "you may…".
- **Use third person only for the framework acting.** "Pest provides…", "Pest will determine…", "Pest automatically…". Pest-as-helpful-agent is a recurring subject.

## Tense & Modality

- **Present tense for behavior and facts:** "The `expect` function returns an expectation instance", "Pest is built on top of PHPUnit".
- **"will" for outcomes and consequences** of the reader's action: "the test will fail", "Pest will display the description", "the result will be returned". The systematic split is *present for capabilities, "will" for results*.
- **"you may" is the dominant modal** — softer and more permission-granting than "you can" or "you must". Reach for it constantly: "you may use", "you may also", "you may wish to". Use "should" for recommendations, never harsh imperatives.

## Sentence Rhythm

- **Medium-length declarative sentences** (roughly 15–30 words) dominate. Rarely terse, rarely runaway.
- **Favor the short-claim-then-qualification rhythm:** a topic sentence stating what something does, followed by a sentence adding a caveat or mechanism, often pivoting on "however" or a semicolon.
- **Reserve short, punchy sentences for the payoff:** "This is game changing." "Pest makes this simple."
- **Keep paragraphs to 1–3 sentences** before a code block. Avoid heavy subordination; prefer chaining short clauses with "However,", "In addition,", "Alternatively,".

## Introducing & Explaining Code

- **Prose almost always precedes code.** The near-universal structure is: one or two sentences of setup → **a colon** → the code block. The colon-into-code rhythm is the single most consistent structural signature of these docs. Even one-liners get a lead-in ("For example, consider the following test:").
- **Show results as code comments,** not prose: `// true`, `// 3`, `// throws InvalidArgumentException`.
- **Narrate complex examples after the code** with "In this example, …" or "As you can see, …".
- **Introduce follow-up variations with a fresh sentence,** then more code: "You may also pass a second argument…".
- **Use concrete, mildly playful example domains** (a `sum` function, a podcast service, a blog) rather than abstract `Foo` / `Bar`.

## Transition & Framing Vocabulary

These connectives are the fingerprint of the voice. Use them, and use them consistently:

- **"Sometimes you may wish to…" / "Sometimes you may need to…"** — the signature opener for optional or edge-case features. Follow with the solution: "To accomplish this, you may…".
- **"By default, … However, you may…"** — the default-then-override pairing. Ubiquitous. State the default, then how to change it.
- **"Of course, …"** — a warm concession or reassurance.
- **"Thankfully, …"** — signals Pest solving a pain point the reader just felt.
- **"Typically, you should…"** — recommends best practice gently.
- **"For example," / "For instance,"** — the standard example lead-in.
- **"However," / "Alternatively, you may…" / "Or, you may…"** — contrast and second approaches.
- **"First, … Next, … Then, … Finally, …"** — sequencing steps.
- **"Once you have…, you may…"** — precondition, then capability.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pestphp/docs](https://github.com/pestphp/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
