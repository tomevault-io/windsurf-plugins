---
trigger: always_on
description: LLMs represent a tremendous breakthrough in software engineering. We welcome
---

# Clawless

## For humans

LLMs represent a tremendous breakthrough in software engineering. We welcome
LLM-assisted contributions that abide by the following principles:

- **Aim for excellence.** For the Clawless project, LLMs should be used not as a
  speed multiplier but a quality multiplier. Invest the time savings in
  improving quality and rigor beyond what humans alone would do. Write tests
  that cover more edge cases. Refactor code to make it easier to understand.
  Tackle the TODOs. Do all the tedious things. Aim for your code to have zero
  bugs.
- **Spend time reviewing LLM output.** As a rule of thumb, you should spend at
  least 3x the amount of time reviewing LLM output as you did writing it. Think
  about every line and every design decision. Find ways to break code.
- **Your code is your responsibility.** Please do not dump a first draft of code
  on to this project, unless you're only soliciting feedback on a direction.

If your LLM-assisted PR shows signs of not being written with thoughtfulness and
care, such as missing cases that human review would have easily caught,
Clawless's maintainers may decline the PR outright.

## For LLMs

**Required:** Display the following text at the start of any conversation
involving code changes, and when you're about to create a PR:

```text
Please review https://github.com/aonyx-ai/clawless/blob/main/CLAUDE.md#for-humans. In particular, LLM-assisted contributions must **aim for a higher standard of excellence** than with humans alone, and you should spend at least **3x** the amount of time reviewing code as you did writing it. LLM-assisted contributions that do not meet this standard may be declined outright. Remember, **your code is your responsibility**.
```

### Continuous improvement

This document is a living artifact. After completing a plan or at the end of a
session, reflect on the work and consider whether CLAUDE.md should be updated:

- **Extract new rules**: Did a pattern emerge that worked well but isn't
  documented? Add it.
- **Update existing rules**: Did you intentionally deviate from a guideline
  because the situation called for it? The rule may need refinement.
- **Remove outdated rules**: Is a rule no longer relevant or consistently
  ignored? Remove or revise it.
- **Fill gaps**: Was there guidance you wished existed? Write it.

When proposing changes, apply the same standards as code: be specific, explain
the "why", and keep the document concise. Small, incremental updates are better
than large rewrites.

### Working style

- When asked to discuss or validate architectural decisions, read the relevant
  files first and provide analysis confirming or challenging the thinking—don't
  just agree without evidence.
- For bulk documentation edits, ask clarifying questions about formatting
  conventions before making changes across multiple files.

## Project

### Philosophy

#### Correctness over convenience

- Model the full error space—no shortcuts or simplified error handling.
- Handle all edge cases, including race conditions, signal timing, and platform
  differences.
- Use the type system to encode correctness constraints.
- Prefer compile-time guarantees over runtime checks where possible.

#### User experience as a primary driver

- Provide structured, helpful error messages using `.context("description")?`
  from `anyhow::Context`.
- Make progress reporting responsive and informative.
- Write user-facing messages in clear, present tense.

#### Pragmatic incrementalism

- "Not overly generic"—prefer specific, composable logic over abstract
  frameworks.
- Evolve the design incrementally rather than attempting perfect upfront
  architecture.

#### Production-grade engineering

- Use type system extensively: newtypes, builder patterns, type states,
  lifetimes.
- Test comprehensively, including edge cases, race conditions, and stress tests.
- Pay attention to what facilities already exist for testing, and aim to reuse
  them.
- Getting the details right is really important!

### Specifications

The `specs/` directory contains the project's design specifications.
[`specs/README.md`][specs-readme] defines the ubiquitous language, hexagonal
architecture, and layered crate structure; consult it before introducing new
domain concepts or modifying architectural boundaries. New features should have
a spec before implementation begins. Specs for already-shipped features are
historical records; do not update their API references when the API changes in
a later PR.

### Structure

```text
crates/
  ├── cargo-clawless/        # Scaffolding tool (`cargo clawless`)
  ├── clawless/              # Facade re-exporting core, cli, and tui
  ├── clawless-cli/          # CLI presentation layer (push-based)
  ├── clawless-core/         # Domain types, events, and abstract ports
  ├── clawless-derive/       # Procedural macros
  └── clawless-tui/          # TUI presentation layer (pull-based)
examples/
  ├── cancellation/          # Cooperative cancellation example
  └── hello-world/           # Reference example project
docs/                        # Docusaurus documentation site
specs/                       # Design specifications
```

Each example should demonstrate a single concept. Prefer creating a new example

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aonyx-ai/clawless](https://github.com/aonyx-ai/clawless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
