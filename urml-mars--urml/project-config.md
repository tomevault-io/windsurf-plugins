---
trigger: always_on
description: <a href="https://urml.dev"><img src="https://urml.dev/favicon.svg" alt="URML" width="72" height="72"></a>
---

<p align="center">
  <a href="https://urml.dev"><img src="https://urml.dev/favicon.svg" alt="URML" width="72" height="72"></a>
</p>

<p align="center">
  A small, opinionated, human-readable language for describing robot intent.
</p>

<p align="center">
  <a href="https://urml.dev"><b>urml.dev</b></a>
</p>

---

# AGENTS.md

> Operational handbook for AI assistants working in this repository (Claude Code, Cursor, Copilot Chat, Aider, any future agent that reads `AGENTS.md` by convention).
> Last updated: 2026-05-23

This file is the working-style companion to [`CLAUDE.md`](CLAUDE.md). `CLAUDE.md` says what URML is and the strategic and architectural rules. `AGENTS.md` says how to act inside the repo: writing style, chat style, outreach discipline, tactical workflow.

**Precedence.** If `AGENTS.md` and `CLAUDE.md` disagree, `CLAUDE.md` wins. If a Claude memory file disagrees with either of these checked-in files, the checked-in file wins. Memory is for state; the checked-in docs are for rules.

---

## Writing style

These rules apply to every artifact a human will see: READMEs, docs, RFC bodies, outreach posts, commit messages, issue replies, this file, [`CLAUDE.md`](CLAUDE.md). They were reinforced after the Hello Robot Stretch 4 outreach packet (2026-05-20: 12 em-dashes in the touch-message body, founder reply: "Too many em-dashes, looks fully machine generated, don't please").

- **No em-dashes** (`—`). Use commas, periods, or parentheses. Replace inline definitions (`X — Y`) with `X. Y.` or `X (Y)` or `X: Y` depending on context.
- **No hedging adverbs:** `importantly`, `notably`, `in essence`, `fundamentally`. Drop them. The sentence is fine without.
- **No throat-clearing transitions:** `furthermore`, `moreover`, `additionally`, `in particular`. Cut them. If you need a transition, start the next paragraph.
- **No empty intensifiers:** `robust`, `comprehensive`, `seamless`, `leverage`, `utilize`, `facilitate`. Use plain words. `use` not `leverage`, `before` not `prior to`, `about` not `regarding`, `help` not `facilitate`.
- **No bold-colon pattern** (`**Foo:** description`) when prose flows fine without it. Bold sentence leads (`**The foo is X.** Then the next thought.`) are fine.
- **No tricolons by default.** Use two items when two will do. Three only when the third earns its place.
- **No over-bulleted text** where short paragraphs work.
- **Short sentences are fine.** Slight informality is fine. Perfect parallelism is not the goal.

Outreach copy especially: write one paragraph at a time, read it aloud, rewrite if it sounds like a marketing email. The recipient is an engineer who clocks LLM copy in two seconds and adjusts trust accordingly.

## Outreach post structure

The Writing-style rules above cover word choice. These cover the shape of a cold outreach post (the Issue or Discussion body a maintainer reads first). They exist because of the Nav2 close: 2026-05-29, Steve Macenski (Nav2 lead maintainer) closed [`navigation2#6184`](https://github.com/ros-navigation/navigation2/issues/6184) saying he tried to answer but the wording was too obtuse to be worth his time. He bounced off the structure, not the topic. A maintainer who has to decode invented vocabulary and pick through seven questions stops reading.

- **Lead with one concrete example, not a description of one.** Show an English sentence, the URML primitive it becomes, and the target's actual call: `move_to(kitchen)` becomes a Nav2 `NavigateToPose` goal. The payload comes first, before any framing about what URML is.
- **One real question. Two at most, never a numbered dump.** If the RFC raises more, they stay in the RFC. The post asks the single highest-value question the maintainer is uniquely able to answer.
- **No invented compound-noun jargon in the post.** If a phrase only parses to someone who already knows URML (`dispatcher-class-only`, `the plugin-set degree of freedom`, `behavior-tree-composition novelty`), it does not belong in a cold post. Say it in plain words or cut it.
- **State up front that the ask is light.** "Apache-2.0, no spec change proposed, nothing for you to maintain." Maintainers triage by how much work you are about to create for them.
- **Link the full RFC as optional depth, never as required reading.** "Full write-up if useful: <link>." Assume the maintainer reads only the post body.
- **A maintainer should be able to read it and answer in under two minutes.** If reading it aloud takes longer than that, it is too long.

The mandatory VIBE disclosure line still goes last (see Outreach identity below), as one line, not a paragraph. The skeleton lives in the `posts-move*` files; the current canonical shape is in [`examples/lighthouses/posts-move17.md`](examples/lighthouses/posts-move17.md).

These rules apply to outreach RFC bodies too, not just the post. Steve called the RFC itself too long and machine-written. An outreach RFC gets the same read-aloud pass: concrete example early, plain language, real drawbacks, no jargon stacks.

## Chat style

The founder writes in very short prompts: `?`, `yes`, `So?`, `No, you do that`. They read outcomes, not narration. Match that energy.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [URML-MARS/URML](https://github.com/URML-MARS/URML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
