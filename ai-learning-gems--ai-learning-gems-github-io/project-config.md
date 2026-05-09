---
trigger: always_on
description: Exact syntax rules for the interactive exercises Quarto extension — required reading before writing or editing exercises
---


# Exercise Syntax Rules

Exact syntax rules for the interactive exercises Quarto extension (`_extensions/exercises/`). These rules are **non-negotiable**: exercises that violate them will silently break (render as raw text, fail to submit, or crash with JS errors).

The extension uses a Lua filter that walks the Pandoc AST. Pandoc parses the markdown *before* the filter runs, so the markdown must produce the exact AST structures the filter expects.

---

## General Rules (All Exercise Types)

1. **Blank lines before and after every exercise block.** The outer `::: {.exercise-*}` and closing `:::` must have blank lines above and below them.

2. **Options MUST use bullet list syntax** (`- Option text`), on consecutive lines with NO blank lines between items. Pandoc parses `- item` as a BulletList node; the Lua filter searches for BulletList. If you use standalone paragraphs, numbered paragraphs, or lettered paragraphs (like `A. text` with blank lines), Pandoc will parse them as separate Para nodes and the filter will not find the options.

```markdown
# CORRECT — consecutive bullet items, no blank lines between:
- Option text one
- Option text two
- Option text three

# WRONG — standalone paragraphs (Pandoc sees Para, not BulletList):
A. Option text one

B. Option text two

C. Option text three

# WRONG — numbered list (Pandoc sees OrderedList, not BulletList):
1. Option text one
2. Option text two
3. Option text three
```

3. **Do NOT prefix options with letter labels** in any format: `A)`, `B)`, `A.`, `B.`, `[A]`, `[B]`, etc. The Lua filter auto-assigns labels A, B, C, D based on list order. These prefixes break in different ways: `A)` triggers Pandoc's ordered list parser producing `<ol type="A">`, while `[A]` renders as literal bracket text inside the option card, duplicating the auto-assigned label.

```markdown
# CORRECT — plain text, labels auto-assigned:
- An agent uses a larger language model
- An agent has a decision procedure that controls a loop
- An agent always uses multiple tools simultaneously

# WRONG — "A)" triggers Pandoc's ordered list parser:
- A) An agent uses a larger language model
- B) An agent has a decision procedure

# ALSO WRONG — "[A]" renders as duplicate label text:
- [A] An agent uses a larger language model
- [B] An agent has a decision procedure
```

4. **The `correct` attribute uses capital letters matching the auto-assigned list order**: first item = `"A"`, second = `"B"`, third = `"C"`, fourth = `"D"`.

5. **Feedback divs are nested inside the outer exercise div.** Each feedback div must have blank lines before and after its opening/closing `:::`.

---

## MCQ (`.exercise-mcq`)

```markdown
::: {.exercise-mcq correct="B"}

Question stem text here.

- First option (auto-labeled A)
- Second option (auto-labeled B — this is the correct one)
- Third option (auto-labeled C)
- Fourth option (auto-labeled D)

::: {.feedback-correct}
Explanation shown when reader picks the correct answer.
:::

::: {.feedback-incorrect}
Explanation shown when reader picks any wrong answer.
:::

:::
```

Required nested divs: `.feedback-correct` and `.feedback-incorrect` (both mandatory).

---

## Prediction Prompt (`.exercise-predict`)

```markdown
::: {.exercise-predict correct="C"}

Prediction question placed BEFORE the surprising result.

- Intuitive-but-wrong option (A)
- Another plausible option (B)
- Correct but surprising option (C)

::: {.predict-reveal}
**The answer is C.** Explanation connecting to the concept.
:::

:::
```

Required nested div: `.predict-reveal` (exactly one).

---

## Ordering / Parsons Problem (`.exercise-order`)

```markdown
::: {.exercise-order correct="C,A,D,B"}

Arrange these steps in the correct order:

- Step text (auto-labeled A)
- Step text (auto-labeled B)
- Step text (auto-labeled C)
- Step text (auto-labeled D)

::: {.order-feedback}
**Correct order: C, A, D, B.** Explanation of why this order is necessary.
:::

:::
```

The `correct` attribute is a comma-separated sequence of letters specifying the correct order. The items in the bullet list should be **scrambled** (not in the correct order).

Required nested div: `.order-feedback` (exactly one).

---

## Fill-in-the-Blank (`.exercise-fillin`)

```markdown
::: {.exercise-fillin}

Stem describing what to complete:

1. Given step one
2. Given step two
3. {B|A: Wrong option text|B: Correct option text|C: Wrong option text}
4. Given step four

::: {.fillin-feedback}
**Correct: B.** Explanation of why B is correct.
:::

:::
```

The fill-in blank syntax is `{CORRECT_LETTER|A: text|B: text|C: text}` inline within a numbered list item. The first element before the first `|` is the correct answer letter.

Required nested div: `.fillin-feedback` (exactly one).

### Fill-in-the-Blank: Critical Restrictions

These restrictions exist because the Lua filter extracts the `{...|...}` pattern from plain text. Anything that interferes with the braces or pipes will break the dropdown.

1. **Do NOT wrap the `{...}` pattern in bold, italic, or any markdown formatting.**

```markdown
# CORRECT:
3. {B|A: Wrong option|B: Correct option|C: Wrong option}

# WRONG — bold wrapping hides braces from the filter:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
