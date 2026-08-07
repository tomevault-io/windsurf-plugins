---
trigger: always_on
description: Guidance for working in this repository. Read this before making changes.
---

# AGENTS.md

Guidance for working in this repository. Read this before making changes.

Zuke is a code-first, strongly-typed build automation system for
Deno/TypeScript.

> This file, `AGENTS.md`, is the single source of truth for both humans and
> agents. `CLAUDE.md` is a thin pointer whose entire content is `@AGENTS.md`, so
> Claude Code loads this file and there is exactly one copy to maintain.

## Using Zuke — the API, without guessing

If you are wiring Zuke into a project, **do not guess the API and do not fall
back to `Deno.Command`/shell.** Every operation has a typed wrapper, and the
exact signatures are published — read them:

- **Check the package catalogue before writing any command.** `llms.txt`'s
  `## Packages` catalogue (raw:
  <https://raw.githubusercontent.com/zuke-build/zuke/master/llms.txt>) and the
  grouped table in
  [`skills/zuke-write-build/references/cheatsheet.md`](./skills/zuke-write-build/references/cheatsheet.md)
  are the only ways to answer "does a `@zuke/<tool>` wrapper exist for this
  CLI?" — per-package `deno doc jsr:@zuke/<package>` can only describe a
  package whose name you already know; it cannot reveal that a package
  *exists*. Reaching for `CmdTasks.exec` (`jsr:@zuke/cmd`) or a raw
  `$`/`Deno.Command` for a tool that has a `@zuke/<tool>` package is a **bug**,
  not a style choice — it discards typed flags, argv purity, and tool
  resolution.
- **One file with the whole typed surface of every package:**
  [`llms-full.txt`](./llms-full.txt) at the repo root. [`llms.txt`](./llms.txt)
  is the short index.
- **A single wrapper on the command line:** `deno doc jsr:@zuke/<package>` (e.g.
  `deno doc jsr:@zuke/deno`).
- **On each package's JSR page / README:** a generated `## API` section.
- **The CLI surface — commands, flags, and a build's actual targets:** run
  `zuke --help` (or `deno run -A zuke.ts --help`). It prints the usage grammar,
  every reserved command (`graph`, `generate-ci`,
  `completions <print|install> <shell>`, `mcp`, `resume`, `runs`, `cancel`,
  `register`, `doc`) and flag, **plus the current build's targets — with
  descriptions and dependencies — and its parameters.** So an agent asked to set
  up or run a build discovers the real command surface live instead of guessing;
  `zuke --list` is the targets-only view and `zuke --list --json` emits the
  whole surface (commands, flags, targets, parameters) as JSON for tools. The
  written reference is [`docs/cli.md`](./docs/cli.md), and
  [`llms.txt`](./llms.txt) carries a generated `## CLI` section; the same data
  is available in code via the exported `describeCli(build)`.

The mental model:

- A build is a class that **extends `Build`**. Each **target is a class field**
  built with `target()`: `.description(...)`, `.dependsOn(...)`,
  `.executes(async () => { … })`.
- **Dependencies are `this.<field>` references, not strings** —
  `dependsOn(this.lint)`, never `dependsOn("lint")` — so renames and typos are
  compile-time errors. A target may only depend on siblings **declared above
  it** (fields initialise top-to-bottom).
- Make the file runnable with **`await run(MyBuild)`** at the bottom — no
  `if (import.meta.main)` guard; `run` no-ops when the module is imported.
- **Every external tool is a namespaced `*Tasks` object** (`DenoTasks`,
  `NpmTasks`, `DockerTasks`, `GitTasks`, …) configured with a **settings
  lambda** that mirrors the real CLI's flags:

  ```ts
  import { Build, run, target } from "jsr:@zuke/core";
  import { DenoTasks } from "jsr:@zuke/deno";

  class CI extends Build {
    lint = target().executes(async () => {
      await DenoTasks.lint();
    });
    test = target().dependsOn(this.lint)
      .executes(async () => {
        await DenoTasks.test((s) => s.allowAll().coverage("cov_profile"));
      });
  }

  await run(CI);
  ```

These three artifacts (`llms.txt`, `llms-full.txt`, and every package's README
`## API` block) are **generated** from `deno doc` by `./zuke apiDocs`, and CI
fails (`./zuke apiDocsCheck`) if they drift — so any change to a public API must
regenerate them in the same PR.

## Tech stack

- **Runtime & toolchain:** [Deno](https://deno.com/) (2.x). All tooling — test
  runner, formatter, linter, type-checker, coverage — is the built-in `deno`
  CLI. No Node, npm, or external build tools.
- **Language:** TypeScript, strict mode (Deno's default).
- **Distribution:** [JSR](https://jsr.io/) as a workspace of 54 packages:
  `@zuke/core` (exports `.`, `./shell`, `./tooling`, `./tooling/conformance`,
  `./render`, `./conformance`) plus the `@zuke/cli` command, a generic `@zuke/cmd` fallback,
  and 50+ typed tool wrappers and plugins (`@zuke/deno`, `@zuke/npm`,
  `@zuke/docker`, `@zuke/ai`, …). The npm org `@zuke-build` is reserved for
  future npm distribution (1:1 name mapping).
- **No runtime dependencies.** The library is dependency-free; tests use a local
  assertion helper (`packages/core/tests/_assert.ts`) rather than a third-party
  assert library so the suite runs with zero network access.

### TypeScript 7 / `tsgo`

The request is to use `tsgo` (the native TypeScript port) **if Deno supports
it.** Status as of 2026-07-30:

- **TypeScript 7.0 shipped (2026-07-08)** and the native Go compiler is no

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zuke-build/zuke](https://github.com/zuke-build/zuke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
