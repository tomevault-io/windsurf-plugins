---
trigger: always_on
description: This file briefs any coding agent -- and any human -- working on
---

# AGENTS.md

This file briefs any coding agent -- and any human -- working on
a10r. It is tool-agnostic: it talks about "the agent", never a
specific product, because the maintainer uses whichever LLM coding
agent fits at the time and may switch whenever. `CLAUDE.md` is a
one-line import of this file; this is the source of truth.

Read this once and hold it throughout a session. For the domain
vocabulary see [`CONTEXT.md`](CONTEXT.md); for the package layout and
end-to-end walkthroughs see [`ARCHITECTURE.md`](ARCHITECTURE.md); for
the point decisions and their rationale see [`docs/adr/`](docs/adr/);
for the contribution mechanics see [`CONTRIBUTING.md`](CONTRIBUTING.md).

## What a10r is

a10r is a fast, intuitive terminal UI for Alertmanager, shaped like
[k9s](https://github.com/derailed/k9s) is for Kubernetes. It targets
SREs, developers, and on-callers who would rather press a key than
click through a web UI. It speaks to both vanilla
[Alertmanager](https://github.com/prometheus/alertmanager) and
[Grafana Mimir](https://github.com/grafana/mimir), is configured from
a single YAML file, and supports multiple tenants -- interacting with
one, several, or all at once. vim motions are the navigation baseline;
`?` opens the help overlay. The TUI is built on Bubble Tea v2's
Model/Update/View foundation (ADR 0042).

## Project framing -- read once, hold throughout

These are maintainer decisions, not preferences to relitigate. They
shape every public-facing choice.

### Pet project

a10r is a spare-time project, built for fun and experimentation. There
is a single maintainer working on a hobbyist cadence -- days, sometimes
weeks per issue or PR. No SLA, no commercial support, no on-call.

- Public-facing copy avoids "we" plurals that imply a team.
- The maintainer's name does not go in the README; the repo URL is the
  attribution.
- No `MAINTAINERS.md`, `GOVERNANCE.md`, `SUPPORT.md`, or `FUNDING.yml`
  -- pet-project framing makes them ceremonial.
- The stale-bot is tuned gentle (90 days to stale, 30 to close), not
  aggressive.
- No launch announcement; let the project be found organically.
- Contributions are welcomed warmly, but do not drum up demand the
  project cannot service. The faster path to a feature is a
  well-tested PR, not a feature request.

### Openly AI-built, tool-agnostic

a10r is openly built with agentic coding -- a human maintainer driving
an LLM coding agent under a strict TDD loop with per-commit subagent
review. This is part of the project's identity, stated plainly, not a
footnote.

- Name the practice (agentic coding, LLM coding agent, AI-assisted),
  never a specific tool as if it were the project's identity. It is an
  agentic-coding project that happens to use whichever agent the
  maintainer prefers at the time.
- Record the tool actually used in commit trailers (`Co-Authored-By:`),
  honestly, per session.
- AI-assisted contributions are welcome on the same terms as any other
  -- DCO, tests, conventional commits, clean `prek -a`. No special
  pleading either way.

### Public contact

The sole public contact address is `a10r@duck.com`. A personal email
address must never appear in any public file, commit author field, or
release artefact. Commit author email stays the GitHub noreply address
(`wilfriedroset@users.noreply.github.com`), which is already
pseudonymous. Before any change reaches a public branch, grep the tree
and confirm no personal address leaked.

## Engineering principles

### No forks

Maintaining a fork is overhead this project does not want. When a
third-party library is missing a feature, compose around it (wrappers,
sibling components rendered alongside) rather than forking and patching.
A fork is considered only after a wrap-or-replace path has been ruled
out and the missing capability is genuinely load-bearing.

### Clean codebase from day one

Favour patterns that keep the code testable and reviewable:

- TDD: write the failing test first.
- Dependency injection: functions take interfaces, structs take their
  dependencies as fields. No globals beyond sentinel errors and
  embedded assets.
- Table-driven tests for anything with input/output fan-out.
- Small, focused units.
- One commit per logical unit. No WIP history, no fix-ups in follow-ups.
  Every commit is `git bisect`-safe -- from v0.1.0 onward. The v0.1.0
  tag is the squash point: the pre-v0.1.0 launch history was collapsed
  into ten narrative phase-commits that are deliberately not individually
  buildable (a feature-area squash does not compile until the tree is
  whole), so `git bisect` is reliable only from v0.1.0 forward.

### Refactor for real, do not game metrics

When a function trips a complexity threshold (gocognit / gocyclo /
cyclop), treat it as a signal to improve the design -- examine the whole
function and its collaborators for the genuine refactor. Do not hoist an
arbitrary slice into a helper just to push the number under the limit.
A cohesive named sub-operation is fine when it removes real duplication
or names a real concept; arbitrary slicing to hide complexity behind a
call boundary is not. Often the right fix changes a collaborator, not
just the flagged function.

## How a change lands

The per-commit review loop is the contract (full version in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilfriedroset/a10r](https://github.com/wilfriedroset/a10r) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
