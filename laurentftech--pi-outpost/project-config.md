---
trigger: always_on
description: <!-- BEGIN OPENLORE (managed — edits inside this block will be overwritten) -->
---

<!-- BEGIN OPENLORE (managed — edits inside this block will be overwritten) -->
<!-- openlore-fingerprint: 25cdd746ebf39b56 -->
This project uses OpenLore for persistent architectural memory.

ALWAYS call `orient()` (via the openlore MCP server, or `npx openlore orient --json`)
before reading source files when starting a new task. This returns the relevant
functions, callers, spec sections, and insertion points for the task at hand —
one structural lookup instead of file-by-file rediscovery.

OpenLore prefixes tool responses with a brief, factual freshness note (the
Epistemic Lease) once your cached context has aged or the repo has moved since
your last `orient()`. It is informational — re-`orient()` if you are relying on
cached cross-module structure; otherwise carry on.

For the MCP setup, ensure `openlore mcp` is configured as an MCP server.
See https://github.com/clay-good/OpenLore for details.
<!-- END OPENLORE -->

## OpenSpec exploration with OpenLore

OpenSpec and OpenLore are independent tools with complementary responsibilities:

- OpenSpec captures intended behaviour, requirements, scenarios, design decisions,
  and implementation tasks.
- OpenLore provides evidence about the existing implementation: entry points,
  ownership, call paths, dependencies, tests, specifications, and likely impact.

When exploring, proposing, reviewing, or updating an OpenSpec change for existing
functionality, call `orient()` early, even if no source file has been opened yet.
Describe both the intended capability and the relevant OpenSpec change when known.

Use the result to:

1. identify existing entry points and responsible modules;
2. find behaviour or constraints missing from the OpenSpec artifacts;
3. detect reuse opportunities, conflicts, and overlap with other functionality;
4. locate relevant tests and existing specifications;
5. estimate the implementation surface and risks.

Reconcile three sources of truth explicitly:

1. the requested intent;
2. the OpenSpec artifacts;
3. the implementation evidence returned by OpenLore.

OpenLore evidence informs the exploration but does not replace requirements or
silently redefine the intended behaviour. When the sources disagree, report the
discrepancy and resolve it in the OpenSpec artifacts before implementation.

For a purely greenfield or conceptual exploration, `orient()` may return little
useful evidence; continue with OpenSpec and state that no relevant implementation
surface was found. If OpenLore is unavailable, use targeted repository inspection
and report the fallback.

## Tooling & CLI Constraints
- ALWAYS use `rg` (ripgrep) instead of `grep` for code search and file inspection.
- NEVER run recursive `grep -r` commands. `rg` is faster and respects `.gitignore`.

## Documentation impact before every PR

Before opening or updating any pull request:

1. Inspect the diff for changes that affect documented behaviour: user flows, CLI
   commands and flags, configuration keys/defaults, environment variables, APIs,
   UI behaviour, deployment, compatibility, prerequisites, or developer workflows.
2. Search every relevant user and developer document (`README.md`, `docs/`, package
   READMEs, examples, and operational guides) for the affected concepts.
3. Compare affected claims, examples, and commands against the implementation,
   schemas, CLI help, package scripts, and tests. Update stale documentation in the
   same PR.
4. Validate changed documentation proportionately: local links and anchors, fenced
   JSON, documented commands, and externally referenced facts where applicable.
5. Add a `Documentation impact` note to the PR description. List the files updated,
   or state `None` with a brief reason when the diff has no documentation impact.

This is an impact review, not a mandatory full reread of unrelated documentation.

## Spec scenario coverage

Before calling a feature complete, prove that every applicable OpenSpec scenario
is covered by testing:

1. Follow the OpenSpec exploration with OpenLore workflow above, then enumerate
   every `#### Scenario:` in
   the relevant main and delta specs. Verify the list with
   `rg '^#### Scenario:' openspec/` so scenarios cannot be silently omitted.
2. Produce an explicit scenario-to-test matrix. Classify every scenario as
   `covered`, `partial`, or `uncovered`, and include the test file and test name.
   Prefer the exact scenario name in the test title or a machine-readable
   `openlore` coverage annotation when the test framework supports it.
3. Read the assertions, not only the test names or suite result. A scenario is
   `covered` only when its GIVEN/WHEN/THEN contract would make the test fail if
   broken. Timing, ordering, negative cases, security boundaries, and observable
   outcomes must be checked at the real boundary described by the spec.
4. Use OpenLore's available coverage, inventory, impact, and spec-drift tools
   when relevant. If an MCP tool is unavailable, use its CLI or `rg` fallback
   and report that fallback.
5. For enumerated surfaces such as routes, commands, or schemas, compare tests
   against the production inventory/source of truth. A hand-maintained test-only
   list is not sufficient proof that newly added cases are covered.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laurentftech/pi-outpost](https://github.com/laurentftech/pi-outpost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
