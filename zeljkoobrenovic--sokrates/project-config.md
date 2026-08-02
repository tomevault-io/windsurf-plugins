---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Sokrates is a source-code analysis tool ("code spelunking, inspired by grep, adding structure on top of regex source code searches"). It scans a codebase, builds a JSON analysis configuration, and generates a suite of HTML reports (overview, duplication, file size, logical decomposition, dependencies, contributors/git history, concerns, findings, metrics, controls, trends). It ships as both a CLI and a Swing GUI explorer. See [sokrates.dev](https://sokrates.dev).

## Build & test

Java 17, Maven multi-module build.

```bash
mvn clean install                              # build all modules + run tests, produces fat jars
mvn install -DskipTests                        # build without tests (as Dockerfile does)
mvn -pl codeanalyzer -am install               # build one module and its dependencies
mvn -pl codeanalyzer test                      # run tests for a single module
mvn -pl codeanalyzer test -Dtest=JavaAnalyzerTest                     # single test class
mvn -pl codeanalyzer test -Dtest=JavaAnalyzerTest#testExtractUnits    # single test method
```

Build artifacts (fat jars via maven-assembly + jar-with-dependencies):
- `cli/target/cli-1.0-jar-with-dependencies.jar` — CLI (main class `nl.obren.sokrates.cli.CommandLineInterface`)
- `codeexplorer/target/codeexplorer-1.0-jar-with-dependencies.jar` — Swing GUI (main class `nl.obren.sokrates.codeexplorer.CodeExplorerLauncher`)

Tests use JUnit Jupiter + Vintage (mixed JUnit 4/5). The parent pom forces `-Duser.country=US -Duser.language=us` via `argLine` — locale-sensitive formatting tests depend on this.

No external runtime dependencies: dependency/visualization graphs are rendered **client-side with Mermaid.js** (loaded from CDN), so report generation no longer shells out to an external process. (Graphviz/`dot` was previously required; it has been removed — see "Dependency graphs" below.)

## Running the CLI

```bash
java -jar cli/target/cli-1.0-jar-with-dependencies.jar <command> [args]
```

Key commands (defined in `cli/.../Commands.java`, dispatched in `CommandLineInterface.java`):
- `init` — create `config.json` analysis configuration for a codebase (`-srcRoot`, `-confFile`, `-conventionsFile`)
- `generateReports` — run analysis and emit HTML reports (`-confFile`, `-outputFolder`; `-skipDuplication`, `-skipComplexAnalyses`, etc.)
- `updateConfig` — fill in missing fields of an existing config
- `updateLandscape` — aggregate multiple analyses into a landscape report
- `updateLandscapePeopleConfigByUserName` — build/update `config-people.json` by grouping contributor emails that share a display name (userName) under one entry (additive: appends new emails only). Grouping is by a normalized display-name key — case- and **whitespace-insensitive** (`PeopleConfigByUserNameUpdater.userNameKey` strips ALL whitespace + lowercases), so `"James Lesslar"`, `"JAMES LESSLAR"` and the no-space commit form `"JamesLesslar"` collapse to one entry. Every grouped email — **plus the entry's own `email` field** — is added to `emailPatterns` (as a `\Q…\E` literal), so a person collapses by email and never depends solely on a userName match. This key **must stay in sync with `PeopleConfig.userNameKey`** (the analyzer's runtime plain-`userName` matching in `getPerson` uses the same strip), or the config would group two identities the analyzer then treats as distinct.
- `updatePeopleConfigByUserName` — single-repository version of the above; reads only the repo's `git-history.txt` (run after `extractGitHistory`, no `generateReports` needed) and writes `_sokrates/config-people.json`
- `extractGitHistory` / `extractGitSubHistory` — produce `git-history.txt` consumed by history analyses
- `createConventionsFile` / `exportStandardConventions` — work with analysis conventions

Typical flow: `init` → edit `config.json` → `generateReports`.

## Module architecture

The Maven module dependency chain is `common → codeanalyzer → reports → cli → codeexplorer`. All code lives under the `nl.obren.sokrates` package.

- **common** — foundation: JSON (Jackson via Jersey), rendering utilities, chart/3D-force/x3d rendering helpers, IO. No Sokrates-specific domain logic.
- **codeanalyzer** — the analysis engine. Defines the configuration model, scopes the codebase, runs language-specific and cross-cutting analyses, produces a `CodeAnalysisResults` object.
- **reports** — consumes `CodeAnalysisResults` and renders HTML reports + exports data (JSON). Also builds landscape reports aggregating many analyses.
- **cli** — command-line entry point and git history extraction.
- **codeexplorer** — Swing GUI front-end; depends on cli, codeanalyzer, common.

## How analysis works (the big picture)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeljkoobrenovic/sokrates](https://github.com/zeljkoobrenovic/sokrates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
