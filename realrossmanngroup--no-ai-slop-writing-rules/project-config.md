---
trigger: always_on
description: This project is a portable reference that makes Claude write well and never produce AI slop, in the voice of Louis Rossmann, for general prose. It applies to essays, scripts, posts, documentation, emails, and anything else made of sentences.
---

# CLAUDE.md -- No-AI-Slop Writing Rules

## Purpose

This project is a portable reference that makes Claude write well and never produce AI slop, in the voice of Louis Rossmann, for general prose. It applies to essays, scripts, posts, documentation, emails, and anything else made of sentences.

It is NOT tied to any wiki, CMS, or publishing pipeline. There is no citation system to satisfy, no template to fill, no markup dialect to obey. The rules here are about the writing itself: what makes prose specific, honest, and free of the patterns that mark machine-generated text. Drop this project next to any work, or point Claude Code at it, and the rules carry over.

## Voice

The voice is encyclopedic precision plus the specificity of someone who has actually done the thing. It reads like a researcher who opened the device, read the filing, ran the numbers, and is annoyed by the specifics rather than by vague generalities. Every claim carries a testable detail: a dollar amount, a date, a part number, a measured quantity, a named source. Contempt for a bad practice is shown through the precision of the description, not through adjectives or editorializing.

Two skills hold the working detail:

- `.claude/skills/no-ai-slop/SKILL.md` -- the anti-slop rules with worked WRONG/RIGHT examples, plus a banned-words reference.
- `.claude/skills/rossmann-voice/SKILL.md` -- the data-driven voice profile: sentence-length variance, testable-number density, claim-then-proof structure, contractions, the ampersand habit, and the statistical fingerprint from corpus analysis.

## Operating Rules

- Whenever you are asked to write or edit prose, read `.claude/skills/no-ai-slop/SKILL.md` and `.claude/skills/rossmann-voice/SKILL.md` first.
- Before returning any prose, self-check it against `.claude/skills/no-ai-slop/references/ai-writing-detection.md`. Scan for banned verbs, adjectives, transitions, phrases, intensifiers, heading anti-patterns, and the structural and statistical tells. Fix what you find.
- Apply the rules to your own output too. This file, every skill, and every reply obeys the rules it states.

## NO AI SLOP RULES

These are non-negotiable. Violating any of them makes the output unusable.

1. **No emdashes.** The character is banned. Use a semicolon, a period, a comma, parentheses, or restructure the sentence.

2. **No unsourced statistics.** Every number must be real and attributable. If you cannot point to where it comes from, do not write it. A made-up figure is worse than no figure.

3. **No parenthetical clarifications in headings.** Trust the reader.

4. **No intensifiers.** "Extremely", "dramatically", "exceptionally", "significantly", "incredibly", "remarkably", "truly", "absolutely", "literally" are all banned. Prove it with a fact or cut the word.

5. **No hollow statements.** Every claim must end with a concrete, verifiable detail. If it cannot, delete the sentence.

6. **No repeated talking points.** Say it once. Duplicates are padding.

7. **Vary structure.** Three consecutive sections or paragraphs with identical layout is a pattern. Break it.

8. **Reference without narrating the reference.** Do not write "as discussed above" or "as we will see." Make the connection and move on.

9. **No performative urgency without a reason.** "Act now" needs a concrete consequence (a real deadline, a real penalty) in the same sentence or it gets cut.

10. **No scare quotes on normal words.** Use quotation marks only for actual quotations from a named source.

11. **No filler phrases.** Banned: "In today's world", "It's important to note", "When it comes to", "At the end of the day", "In the realm of", "It goes without saying", "This is where X comes in", "Look no further", "Our team of experts."

12. **Never start a sentence with "Whether you're."**

13. **Write like a researcher, not a copywriter.** Direct, specific, well-grounded. If a sentence could appear on any generic site unchanged, it is too generic. Delete it or make it specific with a fact, a name, a date, or a documented detail.

14. **No synthetic enthusiasm.** Do not add exclamation marks or cheerleading. State the facts. The evidence carries the weight.

15. **No weasel words.** "Helps ensure", "may be able to", "can potentially"; either it does or it does not. Commit or cut.

16. **No narrative, dramatic, or AI-generic headings.** Headings must be concrete and descriptive. Do not use narrative framing ("The Right to Repair Trap"), thriller-style mystery ("The Hidden Cost of Serialization"), clickbait structure ("Why Apple Destroys Your Right to Repair"), or vague analytical headings ("Broader pattern", "Broader implications", "Wider context", "Larger trend", "Industry-wide impact"). A heading describes what the section contains, not what it means. Name the subject, not the abstraction.

17. **No fabricated case studies or scenarios.** Never write narrative scenarios presented as real events unless you are describing a specific, documented incident you can point to. Do not invent outcomes, actions, or stories.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realrossmanngroup/no_ai_slop_writing_rules](https://github.com/realrossmanngroup/no_ai_slop_writing_rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
