---
trigger: always_on
description: Cutlet is a dynamic programming language built entirely using coding agents.
---

# AGENTS.md for Cutlet

Cutlet is a dynamic programming language built entirely using coding agents.

Cutlet is a dynamic programming language whose long-term goal is to replace Bash for anything beyond trivial one-liners. It aims to provide the expressiveness of languages like Python, Ruby, Lua, and JavaScript while making it dead simple to run subprocesses, build pipelines, handle exit codes, and script your system. Eventually it will excel at parsing text, navigating files and directories, inter-process communication, job control, and quickly building simple user interfaces for one-off tasks — a glue language that can bring together and orchestrate disparate programs. It's optimized for REPL-driven programming, similar to how it's done in Common Lisp or Clojure.

Cutlet is written in C. It's designed to run on Linux and macOS.

## Dependency policy

- **Build requirements**: C23 compiler and POSIX `make`. These are the only hard requirements for building the `cutlet` binary.
- **Libraries (linked into the binary)**: Prefer few, high-impact dependencies. Vendor them in `vendor/` whenever possible (e.g., isocline for line editing). Never shy away from libraries guaranteed to be available everywhere (sqlite, curl, etc.).
- **Dev tooling**: Freely use standard developer tools. The project already uses `clang-format`, `clang-tidy`, and sanitizers. Analysis scripts use `ctags`, `cscope`, and Python 3. These are not required to build or run cutlet — only to develop it.
- **System libraries**: POSIX and platform libraries (pthreads, sockets, etc.) are always fine.

## Limitations of the author

The author of this project is an experienced developer of 20 years. But they don't have experience building and designing programming languages outside of college-level theoretical texts. They also don't have experience with C, primarily being a Rust developer. The author will often need guidance on how best to proceed with the implementation of this language. They will also need an explanation of what the C code is doing.

## Plans and tasks

See `plans/README.md` for project status, key files, and what exists today. Tasks to work on are in `plans/doing/`. Completed work is archived in `plans/done/`. Use `scripts/plan-create <name>` to create a new plan and `scripts/plan-done <name>` to move a completed plan to `done/` with a fresh timestamp.

## Concurrent agent safety

Multiple agents may work on different tasks at the same time. To avoid stale references:

- **Never trust line numbers from plans.** Always search for the target symbol (`grep`, `make symbol-index`, or `Read`) before editing. Plans reference symbols and file paths, not line numbers.
- **Re-orient before each step.** Before starting a plan step, read the relevant files to discover the current state of the code. Another agent may have changed it since the plan was written.
- **Check `make test && make check` after every source code change.** This catches conflicts early — if another agent's edits broke an assumption, tests will surface it. Skip this for non-source changes (documentation, plans, agent config, READMEs, etc.).

## Implementing plans

Plans are implemented inside Docker containers. Each container has the full Cutlet toolchain, Claude Code with `--dangerously-skip-permissions`, and a tmux session. Claude Code inside the container automatically loads `container-claude.md` (copied into the image as `~/.claude/CLAUDE.md` at build time) for container-specific context. Update that file if the container environment or workflow changes.

```
scripts/agent-build                    # build the image (once, or after Dockerfile changes)
scripts/agent-start <branch>           # start a container, attach to tmux
  # inside the container: /cutlet-execute <plan-name>
  # detach with Ctrl+B, D — container keeps running
scripts/agent-connect <branch>         # reattach to a running container
scripts/agent-pause <branch>           # freeze a container when not in use
scripts/agent-list                     # show all containers and their status
  # when done: push the branch from inside the container
scripts/agent-delete <branch>          # remove the container
  # on the host: /cutlet-merge-agent <branch>
```

## Important instructions

- Always write tests first. Include a testing strategy in all plans. All code must be exhaustively tested.
- Before implementing any new code, run `make test` and `make check` to prove that your tests are failing. Pause after test failures and require user confirmation to proceed with implementation. (This only applies when writing source code, not documentation.)
- After editing any C source or header file, run `make format` to auto-format them before running any other checks. This prevents formatting-only failures from `make check`.
- Run `make test` and `make check` after every source code change (C files, headers, Makefiles, test files). Skip for documentation-only changes (plans, READMEs, AGENTS.md, TUTORIAL.md, etc.). **Note:** output from these commands may exceed the tool output limit. Be prepared to pipe through `tail` or `grep` to see the results summary rather than re-running the full command.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s3thi/cutlet](https://github.com/s3thi/cutlet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
