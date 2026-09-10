---
trigger: always_on
description: Read this before starting and go through it again before saying the work is done. It is a summary of the sections further down, which are what each line means in full, and it is the only copy of the summary: the hooks in `.claude/hooks/` pull this section straight out of this file when they run. A line here that disagrees with its section is a bug in this file.
---

# LLM agent instructions for Karl2D

## Checklist

Read this before starting and go through it again before saying the work is done. It is a summary of the sections further down, which are what each line means in full, and it is the only copy of the summary: the hooks in `.claude/hooks/` pull this section straight out of this file when they run. A line here that disagrees with its section is a bug in this file.

When you report on the checklist, write only the items that made you do something: the ones that changed the code, or that you had to act on to satisfy. Say nothing about the rest. A report that walks the whole list buries the few lines that matter in lines nobody needs to read.

- Write procedural, imperative code. A long procedure beats splitting the work across small ones.
- No comment says how the code used to work or what a change improved. The reader has only ever seen the current version.
- Try to keep the diff small: Don't move and reorder whole procedures for no good reason.
- No unrelated code touched, no auto-formatter output, no whitespace changed on lines not otherwise being changed.
- If you need to do breaking changes, then add `@(deprecated)` on old version, if possible.
- Cleanup is written where it happens. `defer` only where several exits would each repeat it.
- A pointer parameter tells the reader the procedure writes through it. Values are automatically passed by reference if big enough, don't pass by pointer to optimize!
- Handles use the zero value `<TYPE>_NONE`, never `Maybe`. Code that runs every frame guards with `!= <TYPE>_NONE`.
- Named return values either drive naked returns, in which case they start with `_`, or say what a returned value is. Never assigned to either way.
- Multi-return results are named `thing_err` and `thing_ok`, never `err_thing`.
- Boxed section comments appear only in `karl2d.odin`.
- Tabs for indentation. At most 100 characters per line in `.odin` files. Markdown files should not have hard linebreaks, we'll use wrapping in editor for those.
- No single line `if` bodies. No spaces in `0..<n` or around the `=` in an attribute.
- Anything that does not fit on one line is split one item per line, each ending with a comma, closing bracket on its own line. Return value lists split the same way.
- Ran the relevant build task(s), and `odin run tools/test_examples` if the change was large.
- If the API surface changed: regenerate `karl2d.doc.odin` with `tools/api_doc_builder`.
- The commit message reads like a tweet: 180 characters at most, simple sentences, the period is the only punctuation.
- The pull request description contains ONE sentence that describes the work in 10 words. After those 10 words comes the things listed under "Testing and reviewing". Put nothing else in the PR description.

## Project Overview
- **Karl2D** is a 2D game development library written in the Odin programming language.
- The focus is on being beginner-friendly, using a minimal set of dependencies and minimizing issues when you actually want to ship the game.
- Karl2D usually requires the latest release of Odin.
- The main entry point is `karl2d.odin`, which contains the platform-independent API and core logic. Platform, render and audio backends live in separate files.
- See `karl2d.doc.odin` for a full API overview. It is generated output: never edit it by hand.

## Workflow
- If the work is on an existing pull request, start with `gh pr checkout <number>` rather than a branch or worktree that merely looks like the right one. It checks out the PR head (including from forks) and sets up tracking, so `git push` updates the PR instead of creating a disconnected branch.
- Keep changes focused. Don't touch unrelated code, don't use auto-formatters (e.g. odinfmt), and don't modify whitespace on lines you aren't otherwise changing.
- If you make unintended changes, revert them in additional commits (squash merges are used).
- If you break backwards compatbility, introduce @(deprecated) procedures or somehow try to do it gracefully. If you cannot help it, then so be, but flag about it in the review.
- Keep dependencies minimal. Prefer clarity and simplicity over cleverness.
- Draft Pull Requests are always welcome and do not need to follow strict rules. A _ready for review_ PR must contain working, tested, complete code that follows the style below.

## Commit messages

Write them like a tweet, max 180 characters. Only simple sentences. Only allowed punctuation is the period. If possible, keep them to 3-4 words. Use more words if really needed.

### The testing and reviewing checklist

Every pull request description contains a list of unticked checkboxes, so the review and the testing can be signed off one item at a time. The boxes belong to the humans who review and test the change. Never tick one, not even for a platform you built and ran yourself, and never tick one later either. The full form looks like this:

```
## Testing and reviewing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karl-zylinski/karl2d](https://github.com/karl-zylinski/karl2d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
