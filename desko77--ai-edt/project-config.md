---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) and other AI agents working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) and other AI agents working in this repository.
Machine-specific paths, ports and workspace names belong in a local `CLAUDE.local.md`, which is
gitignored - keep them out of this file.

## Project Overview

AI-EDT is an Eclipse RCP plugin for 1C:EDT that implements the Model Context Protocol (MCP), so AI
assistants can work against the live EDT model instead of scraping project files. Java 17, OSGi.

- **Plugin ID:** `ru.aiedt.mcp.server`
- **Version:** poms carry the next release as a snapshot - one micro above the newest tag, so a tree
  at `0.2.36-SNAPSHOT` sits on top of `v0.2.35`. The shipped version comes from the release tag
  (tycho-versions rewrites every pom during the release run), so what the tree carries never reaches
  a release; it exists so a local build outranks the release it will replace. Bump it once, right
  after a release. Between releases only the timestamp qualifier moves.
- **Target platform:** EDT 2026.1 (pulled from `edt.1c.ru` by `mcp/targets/default/default.target`)
- **License:** AGPL-3.0-or-later. See `LICENSE`, the per-file headers, and `docs/PROVENANCE.md`.
- **Origin:** the project began as a fork of `DitriXNew/EDT-MCP` and has since been fully
  de-derived; `docs/PROVENANCE.md` records the history and the retained notices.

## Critical Rules

- **ALWAYS BUILD WHEN CODE CHANGES ARE COMPLETE.** After finishing a set of Java changes, run the
  Maven/Tycho build yourself - do not ask for approval and do not defer it.
- **ALL CODE AND INTERFACE MUST BE IN ENGLISH.**
- Before committing, verify `git diff --cached` carries no private data: real project or customer
  names, absolute local paths, IP addresses, credentials, non-default dev ports.
- **A CHANGE IS NOT DELIVERED UNTIL IT IS DESCRIBED AND RELEASED.** When work lands on `main`,
  five things follow before the task is done: the tree version is bumped one micro above the
  newest tag; `README.md` and `README.en.md` say what the user can now do, both or neither;
  `CHANGELOG.md` gains a row for the release, and `docs/tools/README.ru.md` gains the arguments
  a caller has to know; the new tool names, operations and arguments reach **all four** places the
  agent reads them from - the skill under `skills/ai-edt/` in this repository, the skill
  repositories published outside it, the local skill and the local rules, which `CLAUDE.local.md`
  lists by path - in the same pass as the code; and the announcement is written. Updating the
  plugin means updating the rules: a release that leaves one of the four behind is not released.
  Code that only a reader of the diff knows about has not been shipped, and a skill that still
  describes the old behaviour teaches the agent the new one does not exist.
- **PARITY BY FUNCTION, NOT BY NAME.** When adding a capability another plugin also has, implement
  it under our own name at every visible layer - the MCP tool name, the operation name inside a
  facade, and the Java class name. Another product's name is admissible only as a hidden
  compatibility alias, and only when a concrete client depends on it. A collision in a new public
  identifier (tool, operation, response tag, preference key) is a defect to rename, not to ship.
  Rationale: every name-level parity pass regrows structural similarity with the upstream this
  project separated from. Design new mechanisms from our own architecture.

## Branching

Nothing lands on `main` directly. Adopted 2026-09-19; before that, commits went straight to `main`.

- A sprint opens an integration branch `sprint/<version>` off `main` and lives until the release.
- Every feature and every fix gets its own branch off the sprint branch - `feature/<short-name>`,
  `fix/<short-name>` - and merges back into it with a merge commit, so the task boundary stays
  visible.
- A hotfix after a release branches off `main` and merges back into `main` (and into an open
  sprint branch when one exists).
- The sprint branch reaches `main` through a pull request; CI runs on every pull request to
  `main`, and the merge waits for a green run. The release tag is set on the merge commit.
- The tree version bump to the next snapshot goes on `main` after the tag, as before.

## Build System

Maven + Tycho 4.0.5. Modules under `mcp/`:

```
mcp/
  bom/          Bill of Materials, parent POM with build plugin versions
  bundles/      main plugin bundle (ru.aiedt.mcp.server)
  tests/        JUnit fragment of the bundle
  features/     Eclipse feature for installation
  targets/      target platform definition
  repositories/ p2 update site output
```

`build.cmd [EDT_INSTALL_DIR]` autodetects EDT, injects the Directory location into the target, runs
Maven and restores the target. It needs Maven 3.9+ and JDK 17 on PATH.

### Agent build procedure

`build.cmd` is unreliable when driven from an agent shell. Do the three steps manually:

1. **Inject** a Directory location before `</locations>` in `mcp/targets/default/default.target`,
   pointing at a local 1C:EDT component directory. That directory is only a donor for `felix.scr`
   during the build; the target platform itself still resolves EDT from `edt.1c.ru`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Desko77/ai-edt](https://github.com/Desko77/ai-edt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
