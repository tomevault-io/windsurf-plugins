---
trigger: always_on
description: Use when the user explicitly asks to build, extend, validate, repair, rebuild, or merge an LLM-optimized knowledge wiki from markdown notes, documentation, source code, or mixed folders. Default output is a single stable sibling `<source>.wiki/` with full history in a private git repo under `.llmwiki/git/`; `--layout-mode in-place` transforms the source folder itself, and `--layout-mode hosted --target <path>` honours a user-provided `.llmwiki.layout.yaml` contract. SKILL.md is the entry point; 
---


# skill-llm-wiki

## ⚠ Read only SKILL.md and the specific `guide/` files it points to

**Hard rule:** SKILL.md is your entry point. You may additionally read files inside the `guide/` subdirectory of this skill, but **only** those the routing procedure explicitly names for the current operation. Never read anything else in this skill directory: never open `README.md`, never open files under `scripts/`, never open `LICENSE`, never open `guide/` files the routing did not activate, never open arbitrary files out of curiosity. Never `cat` / `head` / `tail` / `Read` a file just to see what's in it.

Scripts under `scripts/` are tools, not references. They are invoked via `node scripts/cli.mjs <subcommand>` exactly as documented in `guide/cli.md`. Never read their source. Never import from them. Never inspect their internals — everything a script does that you need to know is in `guide/cli.md`.

**Why this matters.** Every byte you read is a token spent. This skill is deliberately split so you read only the slice you need for the operation at hand. A typical Validate reads ~18 KB; a typical Build reads ~32 KB; a question about what this skill does reads just this file (~12 KB). Respect the budget.

## ⚠ Preflight: verify Node.js is installed

Before the first `node scripts/cli.mjs` invocation of any operation, run this via the Bash tool:

```bash
node --version 2>/dev/null || printf '%s\n' '__NODE_MISSING__'
```

**Interpret the single line of output:**

- **`__NODE_MISSING__`** — Node.js is not installed. Read `guide/ux/preflight.md`, pick the "Case A" message, and relay it verbatim to the user. **Stop the operation.** Do not attempt any `node scripts/cli.mjs` command. Do not try to install Node yourself.
- **`v<N>.x.x` where N < 18** — Node.js is too old. Read `guide/ux/preflight.md`, pick the "Case B" message, substitute the version string, and relay it verbatim. **Stop the operation.**
- **`v<N>.x.x` where N ≥ 18** — preflight passed. Proceed.

Preflight runs at the start of every operation. Never cache, never skip, never wrap in extra logic. `scripts/cli.mjs` also performs a runtime check and exits with code 4 if invoked on an unsupported Node — defense in depth only; always run the Bash preflight first so the user sees the detailed install/upgrade message from `guide/ux/preflight.md`.

## What this skill does

Builds and maintains **LLM wikis**: filesystem-based knowledge stores structured for deterministic, token-efficient retrieval by a language model. Six operations: **Build**, **Extend**, **Validate**, **Rebuild**, **Fix**, **Join**. Three layout modes:

- **`sibling` (default)** — writes to a stable sibling `<source>.wiki/`. One wiki, one sibling directory, forever. Prior states are reachable as git tags (`pre-op/<id>`, `op/<id>`) in the private repository at `<wiki>/.llmwiki/git/`. No `.llmwiki.v<N>` versioned directory proliferation.
- **`in-place`** — the source folder IS the wiki. `<source>/.llmwiki/git/` is created inside the source; the `pre-op/<first-op>` snapshot captures the user's original content byte-for-byte; rollback restores the original tree exactly. Runs only when the user passes `--layout-mode in-place` explicitly — never inferred, never substituted.
- **`hosted`** — writes to a user-chosen path that carries a `.llmwiki.layout.yaml` contract. The user passes `--layout-mode hosted --target <path>`.

Every operation is a git sequence: `preflight → pre-op snapshot → phase commits → validation → commit-finalize`. Rollback, diff, log, blame, reflog, history, and remote mirroring are first-class skill subcommands under `node scripts/cli.mjs <subcommand>`. Everything is explicit-invocation only — no hooks, no watchers, no automation.

**Ambiguous invocations refuse and prompt.** If the user's request could mean two things (a default sibling would stomp on a foreign directory, a hosted target has no contract, `--layout-mode in-place` is combined with `--target`, …), the CLI exits with code 2 and a structured `INT-NN` error rather than guessing. See `guide/ux/user-intent.md` for the full list.

## Integrating another skill or agent as a consumer

If the user is building a skill, agent, or CI job that calls this skill programmatically (rather than asking you to build a wiki for them in this session), route them to `guide/consumers/index.md`. That subtree answers: how to gate on `format_version`, how to `init` a topic wiki in one command, how to `heal` after every leaf write, how to detect the skill-absent case, how to write consumer tests against the shipped `scripts/testkit/` helpers. Every consumer recipe is dispatched from `guide/consumers/index.md`; do not re-derive the integration path from SKILL.md alone.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ctxr-dev/skill-llm-wiki](https://github.com/ctxr-dev/skill-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
