---
trigger: always_on
description: A new CircleCI CLI built from scratch in Go + Cobra, targeting exemplary CLI design.
---

# CircleCI CLI v2

A new CircleCI CLI built from scratch in Go + Cobra, targeting exemplary CLI design.

Full architecture, command surface, and phased roadmap: `docs/build-plan.md`

> **Branch context:** `main` is the active v1 rewrite. `v0` is the legacy CLI that ships
> today. These are independent codebases — `main` does not import from `v0`. All new feature
> work happens on `main`; `v0` receives only critical fixes.

---

## Critical rules — read before writing any command

These are the six design decisions that must not be violated. They exist because the
current circleci CLI got all six wrong, and this project exists to fix them.

**1. Every data-returning command gets `--json` with field enumeration in `--help`.**
No exceptions. Consistent JSON coverage is the #1 differentiator between a scripting
tool and an interactive-only tool. Use the output helper in `internal/output`.

**2. Use the structured error type in `internal/errors`. Never `fmt.Errorf` in handlers.**
Every error must have: `code`, `title`, `message`, `suggestions[]`, `ref` (doc URL).
Exit code constants live in `internal/errors/exitcodes.go` — always use those, never raw integers.

**3. Never import from the existing circleci-cli.**
This project is a clean rewrite. Importing from the old CLI would carry forward the design
debt we are explicitly replacing. If you need similar functionality, reimplement it here.

**4. `circleci config` = pipeline YAML. `circleci setting` = CLI tool config.**
This naming is non-negotiable. `circleci config validate` validates pipeline YAML.
`circleci setting set token <value>` manages the API token. Never mix these.

**5. Maximum 2 levels of command nesting. If you go to 3, add an alias.**
`circleci context secret set` = fine (2 levels under root).
`circleci job artifacts <n>` = 3 levels → `circleci artifacts` exists as the top-level alias.
The alias is the *primary* user-facing command; the deep path is a thin wrapper that calls into
the same business logic. The alias lives in `internal/cmd/<alias>/` as a full command, not a
Cobra alias string. Four levels must never occur — restructure or alias down to 2.

**6. Every command needs `Use`, `Short`, `Long` (heredoc), and `Example` (heredoc, 3+ examples).**
Examples are "by far the most-read section of help text." Use `github.com/MakeNowJust/heredoc`
for all multi-line strings. No blank `Long` descriptions.

---

## Design guidelines

Full guidelines are in `agents/`. Start with the checklist:

```
agents/checklist.md          ← run through this before any PR
agents/01-philosophy.md      ← the 9 core principles
agents/04-output.md          ← --json, color, TTY detection
agents/05-errors.md          ← error format, exit codes
agents/06-arguments-and-flags.md ← flag naming, short forms, env vars
14-testing.md                ← how to write the tests
```

---

## Package structure

```
cmd/circleci/main.go      Entry point. Cobra bootstrap + top-level error handling.
                          (Lives under cmd/circleci/ so `go install .../cmd/circleci`
                          produces a binary named `circleci`, not `circleci-cli`.)

internal/
├── cmd/                  One package per top-level command (group or alias). Thin Cobra
│   │                     wrappers only — no business logic here.
│   ├── root/             Root command, help topics, global flags.
│   ├── artifacts/        circleci artifacts (top-level alias; primary user-facing command)
│   ├── auth/             circleci auth login/logout/status/token
│   ├── config/           circleci config validate/process/pack/generate
│   ├── context/          circleci context + circleci context secret
│   ├── job/              circleci job artifacts (deep path; wraps internal/artifacts)
│   ├── open/             circleci open (opens current project in the CircleCI web UI)
│   ├── pipeline/         circleci pipeline list/get/trigger
│   ├── workflow/         circleci workflow list/get/cancel/rerun
│   ├── orb/              circleci orb list/info/validate/publish/...
│   ├── project/          circleci project list/follow + project env
│   ├── runner/           circleci runner resource-class/token/instance
│   ├── policy/           circleci policy push/diff/fetch/...
│   ├── setting/          circleci setting list/get/set
│   ├── mcp/              circleci mcp start/stream/tools + editor integrations (claude/cursor/vscode)
│   └── api/              circleci api <endpoint> (raw API escape hatch)
│
├── artifacts/            Business logic for artifact listing and downloading.
│                         Pattern: non-trivial logic lives in internal/<domain>/, not in
│                         internal/cmd/. Commands import from here; never the reverse.
│
├── iostream/             TTY detection, color, stdout/stderr wiring.
│                         NEVER call os.Getenv("NO_COLOR") in a command — ask IOStreams.
│
├── errors/               Structured error type + exit code constants.
│                         exitcodes.go: ExitSuccess=0, ExitAuthError=3, ExitAPIError=4,
│                         ExitNotFound=5, ExitValidationFail=7, ExitTimeout=8
│
├── config/               Read/write ~/.config/circleci/config.yml (XDG standard).
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CircleCI-Public/circleci-cli](https://github.com/CircleCI-Public/circleci-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
