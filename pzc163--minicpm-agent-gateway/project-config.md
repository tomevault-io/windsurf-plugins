---
trigger: always_on
description: MiniCPM Agent Gateway is a local-first Agent routing product built on the Switchyard traffic
---

# AGENTS.md — Switchyard

MiniCPM Agent Gateway is a local-first Agent routing product built on the Switchyard traffic
orchestration engine. It sits between client applications (Claude Code, OpenAI / Anthropic SDK
clients, Codex CLI) and LLM backends, handling routing, format translation, logging, A/B testing,
and health-aware multi-endpoint serving.

> **Compatibility note:** The public distribution is `minicpm-agent-gateway`. Existing imports use
> `switchyard.*`, and the Rust binaries, environment variables, HTTP headers, and metrics retain
> their inherited Switchyard names throughout the 0.1 series.

## Engineering Guidelines

Working principles for any agent (or human) writing code in this repo. These are
about *how* to work; project-specific conventions and validation commands live
elsewhere in this `AGENTS.md`.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```text
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

### 5. Comments Explain Code, Not Project Management

**Source comments are about the code. Tracking lives in the tracker.**

- No issue/PLAN/step references in code (`TODO(step-6)`, "lands in step 4",
  "tracked as ISSUE-001", links to `docs/issues/`). These rot the moment the
  plan changes and leak project-management state into source.
- A plain `// TODO:` describing a concrete code gap is fine; a `// TODO`
  pointing at a tracker step is not.
- Comment what isn't obvious from the code: why a thing is done this way,
  invariants, non-obvious edge cases. Don't narrate what the code already says.
- Module doc comments should state what the module is *for*, not its build
  schedule or its "empty for now" status.

### 6. Commit Discipline

**One step, one reviewed, one-line commit.**

- One focused commit per step; every changed line traces to that step.
- Single-line commit message in Conventional Commits form
  (`type(scope): summary`). No body, no `Co-Authored-By` trailer.
- Pull request titles use the same Conventional Commits form.
- Use `git commit -s` so every commit carries the required DCO sign-off.
- Never commit unprompted. Show the diff, get approval, then commit.

Before repairing DCO, inspect every affected commit:

```bash
git log origin/main..HEAD --format='%h %an <%ae> %s'
```

If every affected commit is yours, add the trailers and update the remote safely:

```bash
git rebase origin/main --signoff
git push --force-with-lease origin HEAD
```

For a mixed-author branch, use an interactive rebase and mark only your unsigned commits for
editing. At each stop, run `git commit --amend --no-edit --signoff`, then
`git rebase --continue`. Never add your sign-off to another contributor's commit.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

### 7. Review Discipline

- Verify every finding against the current code path before reporting it.
- Draft findings before posting review comments unless the user explicitly asks you to post them.
- Resolve only your own review threads, and only after verifying the fix in code.

## Task-Specific Skills

The repository keeps a small set of optional runbooks under `.agents/skills/`. Read a skill only
when its description directly matches the task. Ordinary code exploration, implementation,
testing, and review do not require loading a skill.

| Skill | Use it for |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pzc163/MiniCPM-Agent-Gateway](https://github.com/pzc163/MiniCPM-Agent-Gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
