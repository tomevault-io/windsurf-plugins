---
trigger: always_on
description: **What a person cannot understand is a defect, and so is what a machine cannot
---

# Work on this repository

## What comes first

**What a person cannot understand is a defect, and so is what a machine cannot
read.** A session reports and fixes both like any other defect, and neither is a
matter of taste.

Everything here has both readers, and where the two pull apart the writer serves
both. A name is one a person understands and a check can read. A title scans in
a listing and parses as data. An answer's text says what its schema says. The
rest of this file states what that means in each corner. Where one of these
rules makes something harder to understand, the rule is wrong.

Each of these is a defect on its own. A file name nobody reads at a glance, a
sentence that does not end, a function name with three clauses. A formulation is
short, precise and in plain words. [Prose](#prose) states the rule, and
`bin/cli prose:check` measures it.

Two more rules follow, and each is worth its own statement:

- **Data lives where data lives**, and the writer says it once. A title, a test
  that holds an entry, the version that bounds an answer: each is data. Where a
  reader has to parse one out of prose, somebody writes a regex again and gets
  it right again.
- **Where a person has to guess what something means, that is the defect**, even
  where nothing fails on it. A title that repeats its statement, an id that
  arrives only in a docblock, a file name that claims what the file no longer
  says. Each costs the reader the work the writer saved.

## Layout

```
bin/typo3-dev-companion  # stdio entrypoint; the client launches it as a subprocess
bin/cli            # everything this repository is kept in order by; run it with nothing for the list
src/               # grouped by what a class is: only Paths and the bootstrap sit loose
src/Server/        # starting this server and setting a project up for it
src/Server/Entrypoint.php  # what `bin/typo3-dev-companion` runs: the commands, the usage, the transport
src/Server/Factory.php     # builds the mcp/sdk server from the tool definitions
src/Server/Installer.php   # writes the client setup, publishes the skills, puts back a publication that has gone stale
src/Server/ExcludedTools.php  # what TYPO3_DEV_COMPANION_EXCLUDE_TOOLS takes away
src/Server/CodeAge.php  # whether the running process is older than the code in its checkout
src/Server/Upstream.php  # whether the checkout is behind the repository it comes from
src/Tool/          # one class per tool: its description, its schemas, its answer
src/Tool/Tool.php  # the interface each one implements; ReadOnlyTool carries the annotations
src/Tool/Registry.php  # every tool this server has, and the only place one is switched on
src/Result/        # what a tool hands back, and what several of them build one from
src/Knowledge/     # what this package knows about TYPO3: read from knowledge/, or computed where a rule is applied rather than looked up
src/Knowledge/Scope.php  # which kind of work an answer is for, and how a path is placed in it
src/Knowledge/Coverage.php  # knowledge/server-scope.json: what the server covers, and the initialize instructions
src/Knowledge/Catalog/  # the component catalog and the translation domain derivation
src/Installation/  # what only the installation being read can answer: its icons, labels, namespaces, changelog, project and extensions
src/Installation/Instance.php  # finds the TYPO3 installation the agent is working in
src/Installation/Typo3Cli.php  # runs that installation's console, via DDEV where there is one
src/Installation/Typo3Runtime.php  # boots it in a subprocess and asks its container
src/Installation/probe.php  # what runs over there; never included here
src/Manual/        # the third source: the versioned TYPO3 manuals, and the core changelog above what an installation ships
src/Contribution/  # the fourth source: the services the core's own process runs through
src/Publication/   # the fifth source: the registry a TYPO3 extension is published to, and what it already holds
src/Http/Fetch.php # the one way this server reads a host outside itself: the timeouts, the redirect limit and the agent
src/Search/        # the lexical matching every prose and label lookup goes through
src/Feedback/      # the feedback channel; a development tool, offered from a standalone checkout alone
src/Feedback/Card.php  # the todo a feedback arrives with
src/Sdk/           # the adapters onto mcp/sdk: tool dispatch, the typo3:// resources, and the Skills extension over the skill ones
src/Paths.php      # where this checkout keeps things; the one class both halves share
src/bootstrap.php  # locates the Composer autoloader
src/Upkeep/        # what `bin/cli` runs on this repository, and nothing the server answers with
src/Upkeep/Cli.php # the console application, and the only place a command is switched on
src/Upkeep/Command/  # one class per command, named `<subject>:<verb>` by its own #[AsCommand]
src/Upkeep/Links.php # every path this repository writes between its own files, and whether it still resolves
src/Upkeep/Rst.php   # the reStructuredText the generators write: the underline per level, the directive, the literal, the two roles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TYPO3/dev-companion](https://github.com/TYPO3/dev-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
