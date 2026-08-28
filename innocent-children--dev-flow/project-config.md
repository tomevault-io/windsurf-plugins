---
trigger: always_on
description: Before implementation work, read in this order:
---

# Dev Flow Repository Instructions

## Authority

Before implementation work, read in this order:

1. the user's current explicit request and acceptance criteria;
2. `CONTRIBUTING.md` or `CONTRIBUTING_en.md`;
3. `docs/PRODUCT.md` or `docs/PRODUCT_en.md`;
4. the technical documents directly related to the change;
5. the current source code, schemas, package manifests, and executable tests for the affected surface.

Before a version-only release, read in this order:

1. `release/README.md`;
2. the selected product's release README under `release/`;
3. the current release schemas and publisher contracts;
4. the package manifest and current public-version metadata.

When documentation and executable behavior disagree, use the executable implementation to determine
current behavior and update the affected documentation in the same change. Do not infer requirements
from branch names, directory names, chat history, or historical design documents.

## Requirement Scope

The user's current explicit instruction, current public contracts, and existing product boundaries
define authorized product work.

- Every implementation task must map to the current request, an acceptance criterion, a public
  contract, or an approved engineering constraint.
- Identify the exact files or directories affected before implementation.
- Do not convert rationale, examples, future candidates, or historical incidents into new behavior.
- Do not broaden an implementation because a nearby abstraction appears useful.
- When the request conflicts with current contracts or leaves a material product choice unresolved,
  stop and ask for direction before changing behavior.
- Historical design material is available through Git history; it is not current implementation
  authority.

## Documentation and Internationalization

Human-readable documentation mirrors delivered product behavior; it is not runtime, build, release,
or test authority. The maintained locale set and document-family coverage are defined by
`docs/I18N.md` and `docs/I18N_en.md`.

Every change to user-visible behavior must update documentation in the same pull request:

1. update `README.md` and every maintained root README locale listed in `docs/I18N.md`;
2. update both `docs/PRODUCT.md` and `docs/PRODUCT_en.md`;
3. update each affected technical reference, including `docs/ARCHITECTURE*`,
   `docs/SUPPORT-MATRIX*`, `docs/COMMANDS*`, `docs/ROADMAP*`, host package READMEs, installation
   instructions, or invocation documentation;
4. list the exact documentation paths in the pull-request validation summary.

A version-only release that changes public versions, bundled Core identities, platform support, Host
compatibility, installation commands, or release evidence must synchronize the same facts across all
maintained root README locales and the affected support, command, and package documentation before
publication.

Public end-user installation examples must select the current npm stable channel with
`dev-flow-codex@latest` or `dev-flow-deepseek@latest`. Exact versions must remain in Support Matrix
rows, npm version links, Release Tags, bundled Core identities, artifact digests, and final release
evidence.

Every documented command must be checked against its executable implementation before merge:

- npm package names, `bin` entries, and platform constraints come from the relevant `package.json`;
- Codex subcommands and argument forms come from `packages/codex/bin/dev-flow-codex.mjs`;
- DeepSeek install, inspection, and removal forms come from lifecycle tests and final-artifact
  journeys;
- packaged Core commands come from `cmd/dev-flow/main.go`;
- MCP tool names, annotations, and purposes come from the closed catalog under `internal/mcp/`.

A change that adds, removes, or changes a CLI command, selector, environment variable, lifecycle
command, or MCP tool must update `docs/COMMANDS.md`, `docs/COMMANDS_en.md`, every affected package
README, and all affected root README locale snippets.

- Do not update only one locale when a maintained document family has multiple locale files.
- Do not leave placeholder translations, stale version numbers, untranslated new sections, or an
  English fallback copied into another locale file.
- Preserve commands, identifiers, paths, versions, digests, code blocks, tables, Mermaid graphs, and
  support claims exactly across translations; translate prose, not product facts.
- If synchronized translation cannot be completed, do not report the change as merge-ready.
- A documentation-only correction must update every maintained locale containing the same statement.

## Product Boundary

Only the Go Core owns:

- task and repository-claim identity;
- process definition and content digest;
- current node and resume node;
- action identity and revision;
- node purpose, obligations, allowed effects, and required evidence;
- legal outgoing transitions and transition guards;
- blocker and recovery classification;
- terminal outcome.

Codex, DeepSeek, method tools, CLI, MCP, and package scripts are adapters or execution aids. They must
not persist a second process cursor, add a transition, skip a node, infer completion, or reinterpret a
Core result.

## Method-Tool Boundary


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Innocent-children/dev-flow](https://github.com/Innocent-children/dev-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
