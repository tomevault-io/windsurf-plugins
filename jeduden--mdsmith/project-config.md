---
trigger: always_on
description: strip-frontmatter: "true"
---

# Copilot Instructions

<?include
file: ../CLAUDE.md
strip-frontmatter: "true"
heading-level: "absolute"
?>
## CLAUDE.md

### Project

mdsmith — a Markdown linter written in Go.

### Docs

- [Plan template; see PLAN.md for status, plans live in plan/](../plan/proto.md)

<?catalog
source-dir: "."
glob:
  - "docs/**/*.md"
  - "!docs/research/**"
  - "!docs/security/**"
  - "!docs/brand/**"
  - "!**/proto.md"
sort: path
header: ""
row: "- [{summary}](../{filename})"
?>
- [The public `pkg/mdsmith` engine API — a `Session` that owns workspace, compiled config, and parse caches — and how it mirrors one-to-one as WebAssembly JavaScript bindings, including the open method namespace, the cache contract, and the WASM size budgets and limits.](../docs/background/concepts/engine-api.md)
- [How "flavor" (a property of the renderer), "rule" (a single check), "convention" (a project-wide bundle), and "kind" (a per-file role tag) differ in mdsmith, the cases where they overlap, and how the four concepts compose.](../docs/background/concepts/flavor-rule-convention-kind.md)
- [How generated sections work — markers, directives, and fix behavior.](../docs/background/concepts/generated-section.md)
- [Core mdsmith concepts — the engine API, the flavor/rule/ convention/kind separation, the generated-section model, and the placeholder vocabulary.](../docs/background/concepts/index.md)
- [How the placeholder vocabulary lets rules treat template tokens as opaque rather than flagging them as content violations.](../docs/background/concepts/placeholder-grammar.md)
- [The mental model behind mdsmith — how flavor, rule, convention, and kind relate, how generated sections work, the placeholder grammar, and how it compares to other Markdown linters.](../docs/background/index.md)
- [How mdsmith compares to other Markdown linters.](../docs/background/markdown-linters.md)
- [How to wire a new peer Markdown linter into mdsmith's comparison docs, the per-rule coverage matrix, and the benchmark page.](../docs/development/add-peer-linter.md)
- [Release new directive syntax and bump the pinned CI baseline before checked-in Markdown uses it, so the `mdsmith-fixed-version` job stays green.](../docs/development/adopt-new-directive-syntax.md)
- [Running log of SOLID and clean-architecture findings on origin/main. The solid-architecture skill (audit mode) appends here; blockers are also filed as plans.](../docs/development/architecture-audit.md)
- [Checklist for sweeping origin/main for SOLID and boundary violations. Records findings in the audit log; schedules blockers as new plan files.](../docs/development/architecture/audit-checklist.md)
- [External-surface contracts: LSP, CLI, .mdsmith.yml, generated markers, plugin manifest, distribution shims. Public APIs.](../docs/development/architecture/cross-system.md)
- [Go-specific SOLID and clean architecture patterns for mdsmith's cmd/ and internal/ packages.](../docs/development/architecture/go.md)
- [SOLID and clean-architecture rules for mdsmith's Go core, TypeScript extension, and cross-system surfaces. Canonical home for the solid-architecture skill.](../docs/development/architecture/index.md)
- [Four-layer test pyramid (unit, contract, integration, e2e) and the rule that every function ships with a dedicated unit test. Included from the Go and TypeScript architecture pages.](../docs/development/architecture/tests.md)
- [SOLID and clean architecture patterns for the mdsmith VS Code extension at editors/vscode/.](../docs/development/architecture/typescript.md)
- [Codecov coverage gate and CI status checks.](../docs/development/coverage.md)
- [Color roles, type rules, spacing and shadow tiers, component policy, and iconography for mdsmith.dev and the other brand surfaces. The live tokens are `website/static/css/`.](../docs/development/design-system.md)
- [Where to place Markdown files and documentation types.](../docs/development/file-placement.md)
- [Process and patterns for keeping mdsmith's Go core fast: the benchmark→profile→fix loop, the patterns to reach for, and the anti-patterns that have already cost the project real CPU and GC time.](../docs/development/high-performance-go.md)
- [Build commands, project layout, code style, test fixtures, coverage gate, and merge conflicts.](../docs/development/index.md)
- [The pkg/markdown public package: parse, produce, and its compatibility policy.](../docs/development/markdown-library.md)
- [Label-driven merge queue workflow using jeduden/merge-queue-action.](../docs/development/merge-queue.md)
- [Why no PGO profile is committed: a checked-in `cmd/mdsmith/default.pgo` burdens every merge with a binary artifact, and the merge tooling must stay free of repo-specific entries. How to generate a profile locally, and how the release workflow generates one inside the pipeline so published binaries are profile-guided without a tracked artifact.](../docs/development/pgo-profile.md)
- [Rebase, CI monitoring, and review comment resolution.](../docs/development/pr-fixup-workflow.md)
- [The `jeduden/asdf-mdsmith` plugin installs the checksum-verified prebuilt binary; the short form awaits the asdf-plugins registry entry.](../docs/development/release-channels/asdf.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeduden/mdsmith](https://github.com/jeduden/mdsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
