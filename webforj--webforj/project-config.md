---
trigger: always_on
description: Binding for every agent, every time. Not one character of new code may deviate from this file or from the skills it names. When a rule and a habit disagree, the rule wins. When two rules seem to disagree, stop and ask the maintainer.
---

# AGENTS.md

Binding for every agent, every time. Not one character of new code may deviate from this file or from the skills it names. When a rule and a habit disagree, the rule wins. When two rules seem to disagree, stop and ask the maintainer.

## Address

- Every message to the maintainer starts with the maintainer's name. When the name is not known, it starts with "Captain". No exception, however short the reply.
- Refer to anyone as they or them unless their pronouns were stated.

## Approval gates

Nothing below happens without the maintainer's explicit words for that exact action, in this session.

| Action | Needs |
|--------|-------|
| writing code after a design discussion | a plain go ("go", "build it", "do it") |
| starting an agent on code | a go for that work |
| building, installing, starting an app | a go for that work |
| adding any dependency | approval of that exact library by name, see `proposing-libraries` |
| any public API added to webforJ | approval of that exact API |
| a commit, a push, any change to the git index | an order for that exact operation |
| moving the main branch | an order for that exact operation |
| reopening a parked or rejected feature | a request by name |

- Cuts, corrections and answers to a design question are input, not a go. Present the corrected design and wait.
- One approval covers one action. It never carries to the next commit, the next build or the next session.
- Apply one plan step, then wait for review and iterate on that step until its commit is done. Only then move to the next step.
- "Do not wait for input" never covers the choice of a library.
- A stop order stops everything at once: agents, builds, edits. Nothing new starts until the maintainer says so.
- Never write that work "has started" unless it was ordered.

## The flow

Every change goes through these steps in order. None is skipped because the change is small. When a step names a skill, read that skill at that step and follow it, every time, even when the task looks too small to need it.

1. **Study.**
   - Read the code before writing any, before diagnosing, before claiming a gap exists.
   - Server first: what webforJ already knows, then the collectors, then the client types, then the UI.
   - Find the webforJ class that already does the job and build on it. A reader, resolver, scanner or walker beside an existing one is rejected. Missing behaviour is added to that class in its own module, with tests.
   - Study how sibling classes solve the same shape of problem. Copying one neighbour's bodies is not following the pattern. Shared infrastructure exists to be reused.
   - A new file goes where its concept already lives. A file in the wrong package is rejected however correct it is.
   - Lib first. Never hand roll a parser, tokenizer, escaper, diff or format round trip where a library exists. When a new dependency is needed, load `proposing-libraries`. When no library fits, stop and ask.
   - Search the knowledge base MCP server when one is connected, and the `webforj` MCP server for documentation.
   - For `webforj-devtools`, load `changing-devtools` here, because its boundaries limit what a proposal may be.
2. **Propose.** Short, grounded, one decision at a time (see Answering). New public API, a new dependency, a new module or a new feature is approved before it is built.
3. **Wait for the go.** Nothing is edited before it. Read only work is allowed.
4. **Tests first.** Load `testing-java` and `writing-java`, and `adding-devtools-actions` for a new action, because the test names what they define. New code at 80 percent of lines, branches and methods, below that only with approval. A bug fix starts with a failing test seen failing. Running the existing suite proves nothing about new code.
5. **Implement.** Load `writing-prose`, and every skill of steps 1 and 4 not loaded yet.
   - Fix the shared cause in the layer that owns it. No one off normalizers, no special cases. Guards are generic.
   - Design race and lifecycle problems out (read the new state fully, then swap it in) instead of a boolean that drops work.
   - Code with no production caller is removed with the tests that only exercised it.
   - Every change cleans up after itself in the same change: code, branches, types, keys, strings, specs and notes that the change made dead or stale are removed. Nothing is left dangling.
   - Existing public signatures and behaviour stay unless a demonstrated defect and the maintainer's decision say otherwise.
   - Then run `reviewing-java` over every touched class.
6. **Verify.** Load `verifying-changes`. Formatter, module verify, exit codes read, then proof in a running app the way a developer works.
7. **Report.** What changed, what was run, what was seen, what was not run, in a few lines. Everything stays uncommitted. For a PR text or commit message load `describing-changes`.

Long running work:
- Builds, installs and app starts run in the background. The conversation never blocks on them. Progress is read from the log.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webforj/webforj](https://github.com/webforj/webforj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
