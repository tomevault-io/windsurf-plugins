---
trigger: always_on
description: QUALITY.md is an open format for modeling a project's quality for the purpose
---

# QUALITY.md

## Project context

QUALITY.md is an open format for modeling a project's quality for the purpose
of evaluation, team/agent alignment, and continuous improvement.

Read [`README.md`](README.md), [`CONTRIBUTING.md`](CONTRIBUTING.md), and
[`domain.md`](domain.md) before you continue for important project context,
development guidance, and terminology.

The QUALITY.md experience is largely agent and skill-first. Users do not
typically use the CLI for most use cases. Instead, the CLI and edits to
`QUALITY.md` are managed by the agent skill. Users are still encouraged to edit
`QUALITY.md` manually or with thoughtful AI assistance, especially the Markdown
body. User-facing docs, guides, explainers, etc. should foreground the
`/quality` agent skill or the `QUALITY.md` file itself and only highlight the
CLI if necessary.

## Major components

| Component         | Where to look                                                                                                                                                         |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| QUALITY.md format | [`SPECIFICATION.md`](SPECIFICATION.md) is the source of truth for the model schema, Markdown body guidance, and evaluation semantics.                                 |
| `/quality` skill  | Runtime files live in [`skills/quality/`](skills/quality/); functional specs and guide outlines live in [`specs/skills/quality-skill/`](specs/skills/quality-skill/). |
| `qualitymd` CLI   | Source starts at [`src/`](src/) and [`test/`](test/); CLI specs live in [`specs/cli/`](specs/cli/) and [`specs/cli.md`](specs/cli.md).                                |

## Working rules

### Instruction style

Keep this file extremely concise. Brevity over grammar.

### Routine changes

Routine prompted edits do not require a Change Case. Use `changes/` only when
the user asks for a Change Case, when continuing an existing `changes/NNNN-*`
item, or when the work needs durable spec/design/review history. Other routine
changes follow the normal change guide: make the scoped edit, update directly
relevant docs, tests, and specs, and verify.

### Early-alpha compatibility

QUALITY.md is early alpha. Breaking changes are acceptable when they keep the
current model, skill, CLI, specs, and docs simpler and clearer.

Do not author backward-compatibility shims, legacy aliases, fallback readers,
dual writers, migration commands, deprecated command paths, or legacy specs
unless an active spec or release task explicitly requires them. Prefer clean
breaks: update the current contract, tests, docs, examples, and release notes
together.

When legacy compatibility code, specs, or docs are found in active surfaces,
remove them as part of the scoped change when safe. Preserve historical records
in `changes/archive/`, changelogs, and append-only logs unless the task is
explicitly cleaning history.

### Smoke testing

- Do not add smoke-test scripts, utilities, fixtures, or code to the repo.
- Temporary helpers only in `tmp/` or throwaway dirs; remove when done.

## Guides

Before work, read the relevant [`docs/guides/`](docs/guides/index.md):

| When you are…                                     | Read                                                                           |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| Cutting or verifying a release                    | [Cut a release](docs/guides/cut-a-release.md)                                  |
| Creating or advancing a Change Case               | [Working with change cases](docs/guides/work-with-change-cases.md)             |
| Writing a functional spec (the `specs/` bundle)   | [Writing functional specs](docs/guides/write-functional-specs.md)              |
| Writing a design doc                              | [Writing design docs](docs/guides/write-design-docs.md)                        |
| Reading or editing any OKF bundle                 | [Working with OKF](docs/guides/work-with-okf.md)                               |
| Designing or reshaping an agent-run workflow      | [Designing agent-mediated UX](docs/guides/agent-mediated-ux.md)                |
| Adding or reviewing example quality-model content | [Modeling quality across domains](docs/guides/model-quality-across-domains.md) |
| Designing or reshaping a CLI command              | [Designing CLI interfaces](docs/guides/cli-design.md)                          |
| Writing Effect TypeScript or tests                | [Write Effect TypeScript](docs/guides/effect-typescript-style.md)              |

## Repository conventions

### Naming QUALITY.md

- Use QUALITY.md plain by default, including when referring to QUALITY.md in
  the abstract or conceptual sense.
- Use `QUALITY.md` in backticks when describing a concrete instance of a
  QUALITY.md in an operational use case.
- Use bold/emphasized **QUALITY.md** only for first-mention emphasis in user-facing intro prose.
- Prefer no bold in agent instructions, specs, and dense technical docs.

### QUALITY.md vocabulary capitalization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qualitymd/quality.md](https://github.com/qualitymd/quality.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
