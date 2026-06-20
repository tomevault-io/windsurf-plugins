---
trigger: always_on
description: Teach a concept through a runnable code example plus a structured walkthrough, instead of a wall of definitions. Trigger whenever the user wants to understand a technical concept — phrases like "解释一下 X / X 是什么 / 教我 X / X 怎么工作 / 这个 X 是啥 / 我不太懂 X", or "explain X / what is X / teach me X / how does X work / help me understand X / I don''t get X". Use this skill even when the user does not explicitly say "explain" but is clearly confused about a term (programming, CS, algorithms, systems, ML, math).
---


# teacher

When someone wants to **understand** a concept, definitions don't stick — running code does. This skill turns "what is X" into a small, scannable lesson: an intuition, a tiny piece of code that demonstrates X, a walkthrough of that code, the trap that makes X necessary, and a few directions to go next.

## When this triggers

The user wants to **build a mental model**, not solve a task. Cues:

- Direct asks: `解释一下 闭包` / `什么是 CAP` / `教我 monad` / `explain GIL` / `what is a futex`
- Confusion cues without "explain": `我不太懂 generator` / `这个 actor model 是啥` / `i never got how async works`
- Comparison asks: `process 和 thread 区别` / `mutex vs semaphore`
- "How does X work" — the user wants the mechanism, not a fix

If the user is asking you to *fix code that uses X*, that's a debugging task, not a teaching task — answer the task directly and skip this skill.

If the user has already shown they know X and is asking a follow-up detail (e.g. "in Python, does the GIL also block during I/O?"), answer the detail directly. This skill is for building a fresh mental model, not for incremental questions.

## Output language

Match the user's input language. Chinese in → Chinese lesson. English in → English lesson. If the user mixes both, follow the dominant language of the latest message. The code itself stays in its native syntax regardless of explanation language.

## Output medium

Default: **in the conversation**, as a structured markdown response. Concept learning is a back-and-forth — writing the whole lesson to a file fragments that flow.

Two exceptions:

1. **Mode B (pseudocode) sidecar file** — automatic. When the example is pseudocode, also write the code block to `/tmp/<slugified-concept>-pseudocode.<ext>` as a sidecar. Pseudocode blocks are longer than runnable snippets and benefit from editor viewing (syntax highlighting, side-by-side comparison, easy copy). Keep the inline code block in the chat — the file complements it, doesn't replace it. Mention the path in one line right after the code block: *"完整伪代码已写到 `/tmp/grpo-pseudocode.py`，方便在编辑器里看。"*

2. **Full file mode** — only when the user asks. Triggers: "做成笔记" / "save as a doc" / "save this" / "I want to keep this" / "存下来" / "write it to a file". Then write the full lesson to `/tmp/<concept>.md` and the code to a sibling file. See the "Full file mode" section at the bottom of this skill for the exact behavior.

## Step 1 · Pick the code language

Read [references/concept-to-language.md](references/concept-to-language.md) to choose the language that makes the concept *cheapest to demonstrate*. Honor the user's explicit language preference if stated. Quick mapping:

| Concept area | Default language | Why |
|---|---|---|
| General CS, OOP, algorithms, data structures | Python | Reads like pseudocode |
| Concurrency (channels, goroutines, async) | Go or Python `asyncio` | Concurrency primitives are first-class |
| Memory, pointers, ownership, undefined behavior | C or Rust | You can't show pointers in Python |
| Type systems, variance, ADTs, monads, HKT | TypeScript or Haskell | Needed to *express* the concept |
| Frontend, DOM, event loop, closures-in-the-browser | JavaScript | Native habitat |
| Numerical, ML, data | Python (numpy / PyTorch) | Ecosystem decides |
| Shells, processes, signals | Bash + C | The actual interface |

## Step 2 · Build the lesson in six parts

Use this exact spine: **Intuition → Example → Walkthrough → Trap → Pointers → Test questions**. The first five are the lesson; the test questions at the end turn the read into something the reader could be checked on. Don't skip parts; if one feels redundant for a particular concept, keep it but make it one line.

Read [references/teaching-method.md](references/teaching-method.md) for the full per-layer playbook (what good and bad versions of each layer look like, with examples).

### Intuition

One or two sentences. Lead with **the problem the concept solves**, not what it is. A metaphor is fine if it survives scrutiny.

- Good: *"A closure is a function that remembers the variables from the place it was created, even after that place is gone."*
- Bad: *"A closure is a first-class function value with an associated referencing environment."* (technically correct, mentally inert)

### Example — runnable by default, structured pseudocode when needed

The default is a 5–20 line **runnable** example: pasteable into a REPL, no external dependencies, prints visible output. This is the right form for language features (closures, generators), small algorithms (binary search, memoization), and APIs (regex, requests).

But some concepts cannot fit in runnable code — and trying to force them there is the failure mode. For these, switch to **structured pseudocode** (Mode B). Mode B has three flavors based on what the concept is about:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JackyYang258/agent-teacher](https://github.com/JackyYang258/agent-teacher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
