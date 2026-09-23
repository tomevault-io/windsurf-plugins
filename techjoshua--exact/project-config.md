---
trigger: always_on
description: provides the same confidence with less coupling.
---

# Agent guidance

## Maintainability requirements

All repository changes must follow the
[code maintainability standard](docs/code-maintainability.md). Treat its module ownership, JSDoc,
testing, and change-acceptance rules as required review criteria rather than optional cleanup.

## Automated PR review

CodeRabbit review settings live in [`.coderabbit.yaml`](.coderabbit.yaml). Keep maintenance
rules here and in the maintainability standard; use the YAML for review settings, artifact
filters, and concise guidance about applying those rules. Raw benchmark captures and generated
outputs are excluded from review, while source, fixtures, benchmark runners, and report prose
remain eligible. Review findings require verification against current code before applying fixes.

For new benchmark results, follow the [retention policy](docs/performance-baselines/benchmark-retention.md).
Commit concise results, methodology, environment, source revisions, and derived chart data.
Keep bulk raw captures, traces, logs, copied sources, generated builds, and evidence ZIPs out of Git.
Record dirty-worktree limitations explicitly; a Git SHA alone does not reproduce an uncommitted variant.
Run `npm run check:repository-artifacts` on staged changes. CI also checks each introduced commit,
including files deleted before the PR tip. Do not bypass the artifact allowlist or 1 MiB blob limit;
an intentional exception requires a reviewed policy change with a concrete source or fixture need.

## Keep documentation and agent guidance synchronized

Every feature addition, removal, or behavior change must update all relevant engineering
documentation under [`docs`](docs) and the public documentation application under
[`apps/docs`](apps/docs). Keep in mid the documentation app is not you personal blog. The audience
for the documentation application is external developers and generally it should only explain public
facing features of the framework... unless the page in question is explaining the internal framework
workings intentionally. Also keep in mind that the story page is hand-written content that should
not be changed at all unless it becomes incorrect or misleading, and even then the corrections should
be measured and follow the same tone and style as the underlying document. Treat those updates as part
of the feature, not optional follow-up work. Create, remove, split, combine, or reorganize reference
documents and docs-app pages when that best reflects the resulting framework. Keep proposal status,
current references, docs-app route metadata, navigation, search terms, examples, and stated
limitations consistent with the implemented behavior.

Repository maintenance guidance belongs in this root `AGENTS.md`. Package-local `AGENTS.md` files
are installed-package usage guides for the reusable eXact agent skill, not instructions for
maintaining the package. Keep them brief enough to load without unnecessary context. They should:

- state when an application author should use the package;
- point to the package README for human-readable setup and API orientation;
- name only the safest, most idiomatic usage patterns and important application-facing limits;
- avoid implementation invariants, source-layout notes, test commands, release procedures, and
  instructions addressed to package maintainers.

Package `README.md` files are for human readers. Give them a consistent, professional progression:
identify the package and its purpose, explain when it is useful, show the shortest representative
setup or usage, summarize the important public surface, and link to deeper framework documentation
when needed. Keep them concise and scannable rather than turning them into exhaustive API or design
references. Adapt the sections to the package type: component libraries should show composition,
build adapters should show configuration, runtime libraries should show their main entry point,
and DevTools packages should explain how to install or operate the tools. Internal or
platform-binary packages may use a shorter purpose-and-ownership layout.

Update a package README or local agent guide when its public purpose, setup, recommended usage, or
application-facing limits change. Do not churn either file for an internal implementation change.
Every new package must begin with an appropriate README. Add a concise `AGENTS.md` when the
package exposes an application-authoring surface or the reusable skill otherwise needs
package-specific direction. Ensure published-package manifests include any local guide, and update
the reusable skill whenever a new application-authoring package should be discoverable.

## Keep documentation focused and consolidate completed work

Update the existing document that owns a behavior by default. The synchronization requirement
above does not require a new document for each discovery, implementation step, or internal fix.

- Create a document only when it has a distinct audience and durable purpose that an existing
  reference cannot adequately serve. Explain that need in the change description, not another file.
- Internal refactors normally need appropriate code comments, tests, and a clear commit or PR
  description. Do not create standalone decomposition reports, progress logs, or completion reports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techjoshua/exact](https://github.com/techjoshua/exact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
