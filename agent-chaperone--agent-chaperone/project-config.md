---
trigger: always_on
description: > Repository guide for coding agents and new contributors. Tool-neutral. Point your agent here to get productive quickly.
---

# AGENTS.md

> Repository guide for coding agents and new contributors. Tool-neutral. Point your agent here to get productive quickly.
>
> This is the only guidance file in the repository. Do not add a second one under another name: point the tool at this one instead.

## Project Overview

agent-chaperone is a transparent proxy for the Model Context Protocol. It screens tool calls before an MCP server runs them and tool results before the agent reads them, using deterministic rules plus calibrated judgments from Jev, TypeSafe's System One model. Decisions are probabilities compared against thresholds in a policy file, and every decision is logged.

**License:** Apache-2.0 **Language:** TypeScript (strict), ESM only **Package Manager:** pnpm 10.x **Node:** >= 20.0.0 **Benchmark harness:** Python >= 3.10 with `uv`, under `bench/`

## Repository Structure

```
agent-chaperone/
  src/
    index.ts     Public entry point
    proxy/       Transport plumbing, framing, correlation, and the gate seam
    screening/   The gate: rules, screens, backend and policy in one decision
    cli/         Command line entry point
    screens/     State builders and question batteries
    rules/       Deterministic checks, redaction, hidden text
    policy/      Schema, thresholds, pure decision functions
    backends/    Model backend interface and implementations
    audit/       JSONL writer, and the commands that read it
    approvals/   Single-use tokens that release one held call
    hooks/       Adapter for a client's built-in tools               (#11)
  bench/
    src/         Set builders, runner, scorer
    results/     Recorded model responses and reports
  docs/
    design.md    Architecture, screens, policy, audit, privacy
    hooks.md     Screening a client's own tools, with a worked configuration
    adr/         Architecture decision records
  .github/       CI, release, templates
```

Directories marked with an issue number do not exist yet and arrive with that
piece of work. Traffic is screened end to end and every decision is recorded: `cli/` wraps a
server, `screening/` decides what the relay does with each message, and `audit/`
writes one line per decision. Everything the milestone needs is built; what
is left is the README that first publishes to npm.

## Build Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Compile to dist/
pnpm test             # Run tests
pnpm lint             # ESLint
pnpm typecheck        # tsc --noEmit
pnpm format           # Prettier, write
pnpm format:check     # Prettier, check only
pnpm changeset        # Add a changeset for release
```

Before opening a pull request, run them in the order CI runs them:

```bash
pnpm lint && pnpm format:check && pnpm typecheck && pnpm build && pnpm test
```

CI stops at the first failure, so a formatting problem hides every result after it. `pnpm format:check` only reports; `pnpm format` applies the fixes.

Benchmark:

```bash
cd bench
uv venv .venv && uv pip install --python .venv/bin/python typesafe-sdk
bash fetch.sh                      # download public datasets
.venv/bin/python src/build_sets.py # build labeled sets
.venv/bin/python src/score.py      # score recorded responses, no key needed
```

## Testing Conventions

- Vitest. `*.test.ts` next to the code it tests.
- No test calls the TypeSafe API and no test reaches the network. Model answers come from the fake in `backends/fake.ts`, which replays recordings keyed by a hash of the request. The TypeSafe adapter's own tests drive it through an injected transport.
- The API key is read from `TYPESAFE_API_KEY` by the SDK and by nothing else in this package. Tests set it to an obvious placeholder.
- No test may write to the developer's own state directory. The CLI tests point `XDG_STATE_HOME` at a temporary one, because running the suite should not leave an audit trail of it.
- Proxy paths are tested against a fake upstream server and a fake client. The gate is tested through a real `createProxy` rather than by calling it directly, so ordering, flow control and shutdown are covered by the same tests that cover the decisions.
- Decision functions are pure and tested exhaustively on answer and policy combinations.

## Commit Conventions

Conventional Commits with a body that explains why:

```
feat(screens): add the post-result battery
fix(policy): validate threshold ranges
docs: describe quarantine output
chore: bump dependencies
test(proxy): cover hold and approve
```

Scopes: `proxy`, `screens`, `rules`, `policy`, `backends`, `audit`, `cli`, `hooks`, `bench`.

Branches: `feat/<scope>-<description>`, `fix/<scope>-<description>`, `chore/<description>`, `docs/<description>`, `test/<description>`.

## PR Conventions

- Work from an issue and a feature branch. Never commit to `main`, not for a one-line fix and not for docs.
- One concern per PR. Code, tests, docs, config, and changeset together.
- Stage files by explicit path rather than with `git add -A`, so unrelated work in the tree cannot ride along. `bench/.env`, `bench/data/` and `bench/.venv/` are ignored and stay that way.
- Tests required for the behavior introduced.
- Changeset required when published package behavior changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-chaperone/agent-chaperone](https://github.com/agent-chaperone/agent-chaperone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
