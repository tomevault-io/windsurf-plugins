---
trigger: always_on
description: Deno based CLI for doing AI Native Automation.
---

# Project: swamp

Deno based CLI for doing AI Native Automation.

## Planning

When planning new features, always use the `ddd` skill to inform the
architecture.

## Workflows

In this repository the word "workflow" — including "create/run/execute/validate/
debug workflow", "automate", "orchestrate", and "automated/nightly job" — refers
to a swamp workflow: a declarative YAML DAG of model-method steps authored via
`swamp workflow create`. This is swamp's own first-class concept, implemented in
this codebase, and it is the default meaning here. Load and follow the `swamp`
skill for these requests. Do NOT interpret them as a request to build a Claude
Code agent task list, spin up worktrees, or schedule a cron/remote agent. Only
reach for the harness orchestration tools (TaskCreate/TaskList, EnterWorktree,
CronCreate, RemoteTrigger) when the user explicitly names that mechanism (e.g.
"task list", "subagent", "worktree", "cron", "remote agent") or explicitly asks
you to do the work yourself step by step rather than author a swamp workflow.

## Skills

Skills live in `.claude/skills/<skill-name>/`.

IMPORTANT: Before creating or modifying ANY skill file, you MUST load the
`skill-creator` skill first. Do not skip this step — it contains the
authoritative guidelines for structure, frontmatter, and progressive disclosure.
This is a hard prerequisite, not a suggestion.

Repo-specific rules on top of skill-creator's guidance:

- `SKILL.md` must be uppercase — not `skill.md`.
- After editing any `.md` file in `.claude/skills/`, run `deno fmt` — skill
  markdown follows the same formatting rules as all other files in this
  repository.

After creating or modifying a skill, verify it before submitting:

- `npx tessl skill review .claude/skills/<skill-name>` — quality review of the
  description and content; aim for an average score ≥ 90%. CI enforces that
  threshold for the bundled skills (`swamp`, `swamp-getting-started`) via
  `deno run review-skills`; for other skills it is good hygiene, not a gate.
- `deno run eval-skill-triggers` — promptfoo trigger-routing evals for the
  bundled skills (needs `ANTHROPIC_API_KEY`); run when a bundled skill's
  description or `trigger_evals.json` changed.

See `design/skills.md` for the full skill testing pipeline.

## Code Style

- TypeScript strict mode, no `any` types
- Use named exports, not default exports
- Comprehensive unit test coverage
- All `.ts` and `.tsx` files must include the AGPLv3 copyright header from
  `FILE-LICENSE-TEMPLATE.md` at the top of the file (as `//` comments). Run
  `deno run license-headers` to add headers to any new files.
- No fire-and-forget promises. Every promise must be awaited or explicitly
  handled — unhandled promises race with `Deno.exit` and silently lose data. For
  outbound network calls, pass an `AbortSignal` with a timeout so the caller
  controls cancellation.
- Interpolate values bare in LogTape tagged templates — let the formatter handle
  quoting. Strings passed as `${value}` render as `"value"` in log output;
  wrapping them in literal quotes (`"${value}"`) doubles the quotes to
  `""value""`. Numbers and other primitives render unquoted, so a bare
  `${count}` is correct in all cases.

Changes should only touch what's necessary — don't refactor adjacent code that
isn't part of the task. Keep the blast radius small.

## Commands

Use `deno run` to get a complete list of custom tasks. `deno run dev` runs the
CLI.

## Verification

After completing work, run these checks:

1. `deno check` - Type checking
2. `deno lint` - Linting
3. `deno fmt` - Formatting
4. `deno run test` - Tests
5. `deno run compile` - Recompile the binary

## Source Control & Pull Requests

- Use the `github-pr` skill to create commit messages and pull requests.
- PRs are auto-merged after passing CI and Claude review. To prevent auto-merge,
  add the `hold` label to the PR.
- When a PR fixes a GitHub issue filed by an external contributor (not a repo
  collaborator), add them as a co-author to the commit. Check with
  `gh api /repos/swamp-club/swamp/collaborators --jq '.[].login'` to determine
  if the issue author is a team member. If they are not, add
  `Co-authored-by: Name <email>` to the commit. Use `gh api /users/<username>`
  to look up their name, and use `<username>@users.noreply.github.com` as the
  email unless a public email is available from the API response.

## Architecture

- Follows domain driven design principles. Use the `ddd` skill when designing or
  reviewing code.
- Uses Cliffy for the command line
- Uses Ink for interactive terminal UIs (search, TUI dashboard)
- Uses LogTape for logging and non-interactive output (`"log"` mode)
- Uses JSON for structured output (`"json"` mode via `--json`)
- Every command _must_ support both `"log"` and `"json"` output modes
- You can read the files in `design/*.md` to understand elements of the design

IMPORTANT: CLI commands and presentation renderers must import libswamp types
and functions from `src/libswamp/mod.ts` — never from internal module paths like
`src/libswamp/data/get.ts`. Only libswamp-internal code (other generators, tests
in `src/libswamp/`) may import from internal paths.

## Testing

- Unit tests live next to source files: `foo.ts` → `foo_test.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swamp-club/swamp](https://github.com/swamp-club/swamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
