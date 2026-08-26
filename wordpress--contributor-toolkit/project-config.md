---
trigger: always_on
description: Instructions for AI coding agents working in this repository. Tool-neutral and canonical — `CLAUDE.md` points here rather than repeating it.
---

# AGENTS.md

Instructions for AI coding agents working in this repository. Tool-neutral and canonical — `CLAUDE.md` points here rather than repeating it.

## Where things are, whatever agent you use

Two files, and neither is specific to one tool despite what their paths suggest:

**The review standard** — [`.github/instructions/code-review.instructions.md`](.github/instructions/code-review.instructions.md). The five dimensions, this project's invariants, the procedure to run them, and the reporting format. Read it directly if your agent has not already.

It sits under `.github/instructions/` because Copilot code review reads that directory natively and follows no links out of it. Anywhere better-named would have meant maintaining a condensed second copy for Copilot, which would drift. Every other agent reaches it from here.

**The review as a skill** — a thin `SKILL.md` wrapper over the file above, in two locations because no single skills directory is read by every agent:

- [`.claude/skills/self-review/`](.claude/skills/self-review/) — Claude Code and Copilot. In Claude Code it is `/self-review`.
- [`.agents/skills/self-review/`](.agents/skills/self-review/) — the Agent Skills open-standard path that Command Code and others scan.

Both are pointers to the one instructions file, not copies of the standard. If your agent looks somewhere else again (`.github/skills/`, `.cursor/skills/`, `.codex/skills/`, or its own convention), add a wrapper there or skip it entirely and follow the instructions file — that is where all the content lives. The wrapper only adds two things: run the judgement pass in a fresh context, and report without touching GitHub.

In Codex, `/review` is the built-in user-facing review command. Whenever `/review` runs, read and follow [`.github/instructions/code-review.instructions.md`](.github/instructions/code-review.instructions.md) in full as the review standard. `self-review` is the local skill name, not a Codex command; do not tell users to invoke `self-review`.

There is deliberately no per-tool copy of the *standard*. A wrapper is a few lines that defer to it; duplicating the standard itself is the thing to avoid — if you find yourself doing that, fix the pointer, not the number of copies.

## What this is

An Electron desktop app ("WordPress Contributor Toolkit") that sets up a full WordPress core (`wordpress-develop`) dev environment with zero prerequisites — no Git, Node, npm, or Docker required on the host. Everything is bundled and run as JS/WASM inside the Electron process. Built to fix a Contributor Day problem: newcomers burning the whole session on local setup instead of contributing. Still labeled "experimental."

## Before opening a pull request

Run the review in `.github/instructions/code-review.instructions.md` against the branch, and fix or consciously defer every finding. Summarise the outcome in the pull request description — counts, what was fixed, what was left as a follow-up and why.

Before reporting a GitHub workflow complete, verify every requested final state on GitHub — for example, distinguish a merged pull request from one that is merely closed, and confirm that an issue was closed by the intended pull request rather than only by a comment.

Nothing enforces this. There is no automated review on pull requests, by design: it would mean storing an AI provider credential as a secret in a public repository. This pass is what stands in its place, so skipping it means a human reviewer is the first reader of the diff.

That file carries the procedure as well as the standard. Follow it rather than improvising a review.

### The pull request description follows the template

[`.github/pull_request_template.md`](.github/pull_request_template.md) is the shape, and GitHub loads it into every new pull request automatically — including ones opened with `gh pr create`, as long as you do not pass a `--body` that replaces it. Fill it in rather than writing your own structure.

The rule it is built around: **a reviewer understands the change in five minutes.** So what stays visible is Why, What changes, How to test this, Risks, Related — and everything else goes in a `<details>` block, collapsed by default. Depth is not the enemy of a readable PR; depth *in the way* is. Do not delete detail to hit the five minutes, move it.

**One template, not one per kind of change.** GitHub shows no picker when a pull request is opened — selecting among several requires appending `?template=name.md` to the URL, which nobody remembers, so the default loads anyway. The template is written to serve a fix, a feature and a process change equally, and calls out the three places where they genuinely differ: a fix names its root cause and the test that fails without it, a feature names what it deliberately leaves out and shows its surface, and either way the testing steps follow the path a contributor actually takes.

That includes the review outcome AGENTS.md requires below: it lives in a collapsed block, with the headline count surfaced in **Risks and limitations** when it changes how the PR should be read.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WordPress/contributor-toolkit](https://github.com/WordPress/contributor-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
