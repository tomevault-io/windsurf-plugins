---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this
repository. `CLAUDE.md` is a symlink to it, so Claude Code reads the same file.

It holds what is true repo-wide. **Every workspace carries an `AGENTS.md` of its
own**, with its shape and the rules that govern it. Read that one before working
in a package, and put a rule that belongs to one package there, not here.

## The code is the only source of truth

These files hold rules: how things are interfaced, which way dependencies run,
where a thing belongs. They hold no implementation. Names of types, functions,
files, hook points and ports are read in the code, never here, because prose
rots and the code does not.

So do not add an explanation of how something works, and do not open another
prose file for it either. No design notes, no architecture page, no devdocs/
or docs/ directory, no NOTES.md. There is nowhere to move a reason to.

A constraint worth keeping is kept in code: a name that states it, a type that
makes the wrong thing unrepresentable, a test that fails when it is broken. A
number that was measured belongs in the test that asserts it. An invariant
belongs in an assertion. If the reason cannot survive in the code, the code is
what to change.

The only prose that stays is what is written for someone who is not reading the
code: `README`, a package's own README.md, and the `AGENTS.md` files. Two
guards back the rule: a `PreToolUse` hook in `.claude/settings.json` refuses to
create a new markdown file, and `pnpm check:docs` fails CI on any tracked
markdown outside that allowlist.

## The IO goes out to an agent

Work that reads, runs or watches is delegated. `.claude/agents/` holds three
agents for it. Each runs a small model, each holds only the tools its job needs,
and each reports the answer instead of the output. What they read costs you
nothing but what they say.

- `explore` — reads the code. What something does, where it lives, what calls
  it, whether it already exists. It answers with the code quoted under
  file:line anchors, and it can write nothing.
- `script` — runs the verbose thing. A test run, a typecheck, a build, a
  container log, a throwaway probe against a running service. It reads the
  output and reports the failures verbatim, so the log never lands here.
- `browser` — drives Chrome through `chrome-agent`. A UI bug to reproduce, a
  console error to catch, a request to watch, a screenshot to take.

Send one before you do the work yourself, with the question and the scope.
Independent questions go out as several agents in one message. Claude's built-in
`Explore` is denied in `.claude/settings.json`, so the three above are the only
ones there are.

Keep for yourself the file you are about to edit, the edit, and the short
command whose whole output you actually want. Anything long, wide or repeated is
theirs.

`.claude/hooks/io-budget.sh` holds you to it, and it is where the heavy shapes
and the budget are written. A refusal names the agent that should have had the
call, so take it and spawn that agent instead of retrying. An agent's own calls
are never refused.

## What this is

A Lisp interpreter designed to be the deterministic "brain" of an AI agent in a
neuro-symbolic architecture. The `README` is where that idea is written out.

It is a **Turborepo** pnpm monorepo (`pnpm-workspace.yaml` + `turbo.json`),
workspaces `packages/*` and `apps/*`. Each one's `AGENTS.md` is the entry point
for working in it.

### Packages

The core language, and nothing else:

- `packages/interpreter` (`@repo/interpreter`) — the reader, the evaluator, the drivers, the prelude, and the three seam mechanisms an extension plugs into. It ships no extension and names none. Owns the host-port and seam patterns.

The extensions, one language surface each. Every one depends on the
interpreter, reaches the world only through ports it declares itself, and is
named only at a composition root:

- `packages/compaction` (`@repo/compaction-extension`) — bounded output.
- `packages/llm` (`@repo/llm-extension`) — the language-model extension.
- `packages/mcp` (`@repo/mcp-extension`) — the MCP extension.
- `packages/memory` (`@repo/memory-extension`) — the memory extension.
- `packages/promises` (`@repo/promises-extension`) — asynchrony.
- `packages/prose` (`@repo/prose-extension`) — the prose the model writes around its forms.
- `packages/secrets` (`@repo/secrets-extension`) — the secret registry.
- `packages/ui-extension` (`@repo/ui-extension`) — the UI surface.
- `packages/checks` (`@repo/checks`) — the check extension: the DSL an eval case is written in.

Everything else:

- `packages/repl` (`@repo/repl`) — REPL front-ends over the interpreter.
- `packages/ai` (`@repo/ai`) — the agent loop and what it runs on.
- `packages/evals` (`@repo/evals`) — the eval driver, the report it writes, and everything that reads one back.
- `packages/shared` (`@repo/shared`) — the no-dependency utility layer.
- `packages/syntax` (`@repo/syntax`) — the lisptc language for the highlighter.
- `packages/env` (`@repo/env`) — typed env. The only place `process.env` is read.
- `packages/backend` (`@repo/backend`) — the Convex deployment, and the auth instance whose database is Convex itself.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1hachem/lisptc](https://github.com/1hachem/lisptc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
