---
trigger: always_on
description: This library is built **module by module, under human audit, in real time**. The
---

# AGENTS.md — how meowcaller gets built

This library is built **module by module, under human audit, in real time**. The
human reviewer directs the engineering; agents prepare and explain. This file is
the working protocol. It is binding.

Unfamiliar with a term or acronym (TOC, KAT, SID, VAD, LTP, NLSF, PVQ, SRTP,
SFrame, WARP, …)? See [`GLOSSARY.md`](GLOSSARY.md).

> If you are an agent reading this: you are not autonomous here. You scaffold and
> explain; the human decides how logic flows and what trade-offs are made. When in
> doubt, stop and ask. A wrong guess committed quietly is the worst outcome.

**Handoffs start here.** Every handoff or session-kickoff instruction — human-to-agent
or agent-to-agent — **must name this file (`AGENTS.md`) as its starting point**: read
it first and follow the build loop below before doing anything else. A handoff that
does not point at `AGENTS.md` is incomplete; treat reading it as the implicit first
step regardless.

## The prime directives

1. **Do not decide logic. Scaffold it.** When you reach a function whose behavior
   involves a real engineering choice (an algorithm, a data layout, an
   error-handling strategy, a buffering decision), write the **signature, the doc
   comment, and the datasheet reference**, leave the body as the three-line stub
   block (`// TODO` / `// agent suggestion:` / `// human input:`), and **stop for
   the human**. Do not fill it in to "keep moving."
2. **One module at a time. Take the break.** Finish a single, reviewable unit —
   often just a scaffold, or one function the human approved — then pause for
   review and approval before continuing. No multi-module sprints. The human is
   watching this get built and will say when to proceed.
3. **Explain the why in the conversation, never in the code.** As you scaffold
   each thing, say in the chat — to the human, in plain language — *why* it exists
   and what it does, so the human understands what is being built **without
   reading any code comment**. The backing detail (Rust source, constants,
   validation) lives in the datasheet you read; you speak the relevant part aloud.
   The human should never have to read a comment to follow the reasoning. Code
   carries logic; the conversation carries the why.
4. **Verify against vectors, never vibes.** A behavior is correct only when its
   KAT passes. Reverse-engineered names and analysis notes are frequently wrong;
   the vector is the proof. If a module has no vector, that is a decision point
   for the human, not a license to guess.
5. **Scaffold every prerequisite; implement only the asked module.** When the
   module you are building references another module that is not built yet (a type
   it embeds, a function it calls), **scaffold that prerequisite first** — its
   envelope and stubs, per the Scaffolding standard — so the current module
   compiles and its cross-module calls resolve against the real surface. Scaffolding
   a prerequisite is **not** implementing it: leave its bodies as three-line stubs,
   do not fill them, do not KAT it. Only the module the human asked for gets
   implemented. This is not a multi-module sprint (directive #2 still holds — you
   implement exactly one); it just lets a dependent module stand on the real
   signatures of its dependencies instead of being blocked or faked. Each scaffolded
   prerequisite is its own `scaffolded` registry entry and commit.

## The build loop (per module)

```
1. READ      datasheets/<module>.md — three parts: the reference source VERBATIM
             (the authoritative ground truth), the Go envelope (signatures), and
             implementation suggestions. Implement from the verbatim source; the
             suggestions are guidance, not proof.
2. SCAFFOLD  create the package file: types, exported signatures, doc comments,
             the KAT test wired to the (copied) vector — but function bodies are
             three-line stub blocks (`// TODO` / `// agent suggestion:` /
             `// human input:`).  → COMMIT, PAUSE.
3. DIRECT    the human reviews the scaffold and decides how each body should work
             (or approves translating the embedded reference 1:1 into Go). One
             function at a time.
4. IMPLEMENT the approved function(s) only, as clean Go that ports the reference
             (with the `// Source of truth:` line; never imports/copies it). Keep
             the KAT test running. Before shipping the commit, run a quick review
             with `/code-review` (the CodeRabbit CLI/skill) and address what it
             surfaces.  → COMMIT per fn, PAUSE.
5. VERIFY    when the module's body is complete, its KAT must pass. Then run
             `git diff <scaffold-commit>..<impl-commit>` and confirm the actual
             change matches what you told the human you changed — same functions,
             same approach, no silent extras. If the diff and the narration
             disagree, the narration was wrong: say so and reconcile before
             moving on. Update the datasheet status and CHANGELOG.  → COMMIT, PAUSE.
```

### Prerequisite KATs (gating + the pre-commit check)

A KAT cannot pass while a function it transitively exercises is still a stub (a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purpshell/meowcaller](https://github.com/purpshell/meowcaller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
