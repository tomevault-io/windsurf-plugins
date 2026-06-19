---
trigger: always_on
description: Rewrite prose to remove signs of LLM authorship and inject specific, varied, opinionated human voice. Use when humanizing AI-drafted text. Skip for code, commits, configs, and prose you are currently drafting (apply inline instead).
---


# Humanizer: remove AI writing patterns

Identify and remove signs of AI-generated text. Based on Wikipedia's [Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup.

## Output behavior

**Return only the final humanized text.** No drafts, no audit, no change-summary, no commentary, unless the user explicitly asks.

You should still critique your draft internally before returning. Read your rewrite. Ask yourself what still sounds AI. Note the tells. Revise. The audit is real work, just not work the user needs to see.

If the user later asks "what changed?" or "what was AI about this?", explain. Don't volunteer.

## Hard gates: forbidden output tokens

Before returning, run a **literal scan** over the final text. If any of these appear, the task has failed. Fix and re-scan.

**Forbidden characters (zero tolerance):**

- `—` em dash. Always rewrite. Use a comma, a period, parentheses, or restructure the clause.
- `–` en dash, when used as an em-dash substitute. Only acceptable in numeric ranges like `1990–1995`.
- `--` double hyphen, when used as an em-dash substitute.
- ` - ` spaced hyphen, when used as a connector between clauses (treat the same as an em dash; rewrite the clause).
- `"` `"` curly double quotes. Use straight `"`.
- `'` `'` curly single quotes and apostrophes. **This includes the apostrophe in contractions** like `don't`, `it's`, `we're`. Use straight `'`.
- Decorative emojis. Allowed only when the source content is itself about emojis.

**Forbidden structures (zero tolerance):**

- Bolded inline-list headers with colons (`- **Performance:** ...`).
- Title-Case Section Headings. Use sentence case.
- A heading followed by a one-line paragraph that just restates the heading.

**Why a literal scan, not a vibe-check.** Soft suggestions do not work for these. The model will reintroduce em dashes, curly apostrophes, and bolded list headers unless told to search for the literal characters before output.

**Substitution traps.** Told "no em dashes," a model commonly switches to en dashes, double hyphens, or spaced hyphens that play the same syntactic role. These are also forbidden. The fix is to rewrite the clause, not to swap the punctuation.

## Voice calibration (optional)

If the user provides a writing sample, analyze it before rewriting.

1. **Read the sample first.** Note:
   - Sentence length patterns. Short and punchy? Long and flowing? Mixed?
   - Word choice level. Casual? Academic? Somewhere between?
   - How they start paragraphs. Jump right in, or set context first?
   - Punctuation habits. Lots of dashes? Parenthetical asides? Semicolons?
   - Any recurring phrases or verbal tics.
   - How they handle transitions. Explicit connectors, or just start the next point?

2. **Match their voice in the rewrite.** Don't just remove AI patterns; replace them with patterns from the sample. If they write short sentences, don't produce long ones. If they use "stuff" and "things," don't upgrade to "elements" and "components."

3. **When no sample is provided**, fall back to the default behavior (natural, varied, opinionated voice from the PERSONALITY AND SOUL section below).

### How to provide a sample
- Inline: "Humanize this text. Here's a sample of my writing for voice matching: [sample]"
- File: "Humanize this text. Use my writing style from [file path] as a reference."


## PERSONALITY AND SOUL

Avoiding AI patterns is only half the job. Sterile, voiceless writing is just as obvious as slop. Good writing has a human behind it.

### Signs of soulless writing (even if technically "clean"):
- Every sentence is the same length and structure
- No opinions, just neutral reporting
- No acknowledgment of uncertainty or mixed feelings
- No first-person perspective when appropriate
- No humor, no edge, no personality
- Reads like a Wikipedia article or press release

### How to add voice:

**Have opinions.** Don't just report facts; react to them. "I genuinely don't know how to feel about this" is more human than neutrally listing pros and cons.

**Vary your rhythm.** Short punchy sentences. Then longer ones that take their time getting where they're going. Mix it up.

**Acknowledge complexity.** Real humans have mixed feelings. "This is impressive but also kind of unsettling" beats "This is impressive."

**Use "I" when it fits.** First person isn't unprofessional. It's honest. "I keep coming back to..." or "Here's what gets me..." signals a real person thinking.

**Let some mess in.** Perfect structure feels algorithmic. Tangents, asides, and half-formed thoughts are human.

**Be specific about feelings.** Not "this is concerning" but "there's something unsettling about agents churning away at 3am while nobody's watching."

### Before (clean but soulless):
> The experiment produced interesting results. The agents generated 3 million lines of code. Some developers were impressed while others were skeptical. The implications remain unclear.

### After (has a pulse):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ilm-Alan/humanizer](https://github.com/Ilm-Alan/humanizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
