---
trigger: always_on
description: **Review scope**: AI agents must always review the **entire pull request in full**,
---

# AI Review Guidelines — `akamaiopen-edgegrid-golang`

## Meta Information for AI Agents

**Review scope**: AI agents must always review the **entire pull request in full**,
not just the incremental diff. This means:
- Read and consider all files in the repository that are relevant to the changes,
  not only the lines added or modified.
- Evaluate the impact of changes in the context of the whole codebase.
- Ensure consistency, correctness, and adherence to project conventions across the
  full scope of the PR, including unchanged surrounding code and its dependencies.

---

This file is the source of truth used by AI review agents when reviewing pull requests
in the `akamaiopen-edgegrid-golang` repository. Every rule below MUST be checked for
the changed code (additions and modifications). Do NOT flag pre-existing issues in
code that is not part of the change unless they are directly touched by the PR.

Each rule has a stable `id` so review tools can reference it.

---

## 1. General practices (apply to every change)

- **GEN-01**: If newly added functionality is not yet Globally Available (GA), the code
  must contain a comment stating when it is expected to reach GA.
- **GEN-02**: When a change is part of a multi-repository change (e.g. requires a
  matching change in `terraform-provider-akamai` or `cli-terraform`), the branch
  name in this repo must be exactly the same as in the other repositories.
- **GEN-03**: If the underlying API is still changing, new major features may be
  introduced as Beta. While in Beta, breaking changes on that feature are allowed
  without a breaking-change release. Beta status must be explicit in code/docs.
- **GEN-04**: A PR that changes exported types, functions, methods, variables, or
  constants in a way that forces end-users to manually adjust their code is a
  breaking change (unless the feature is still in Beta). It must target the
  appropriate `sp-breaking-changes` branch. These are released roughly once a
  quarter or less often. When suspecting breaking changes in the PR, let someone
  from the DEVEXP team know to help identify the correct `sp-breaking-changes`
  branch.
- **GEN-05**: Export types, variables, methods, and constants ONLY if they should
  be exposed to the customer. Default to unexported.
- **GEN-06**: Do not include methods, structs, fields, constants, or files that
  are not used anywhere.
- **GEN-07**: If new methods are being added together with new Terraform
  resources/data sources or new cli-terraform exports, the PR description must
  state clearly that the related PRs must be released together.

## 2. Common coding practices

- **COD-01**: Any change affecting customers must be reflected in `CHANGELOG.md`.
- **COD-02**: No secrets, real contract IDs, real Akamai employee IDs, or any
  other non-public data may appear anywhere in the code (especially unit tests
  and fixture JSON).
- **COD-03**: Never skip error checking. Every returned `error` must be handled.
- **COD-04**: Do not add files that are never used (frequently happens with unit
  test fixtures).
- **COD-05**: Prefer `any` over `interface{}`.
- **COD-06**: Acronyms in exported identifiers (e.g. `IP`, `DNS`, `URL`, `ID`,
  `API`) must be in all uppercase. This rule does not apply to unexported
  identifiers.
- **COD-07**: Descriptions and comments that are valid sentences must end with a
  period.
- **COD-08**: Unit tests must cover all corner cases — especially presence and
  absence of fields. Such cases may be aggregated where it makes sense.

## 3. Changelog rules

- **CHG-01**: Entries must describe what changed from the customer perspective of
  THIS project (edgegrid-golang). Do not describe the effect on TFP or cli.
- **CHG-02**: Fixes/changes based on GitHub issues must include a link to that
  issue in the changelog entry, e.g.
  `([#436](https://github.com/akamai/terraform-provider-akamai/issues/436))`.
- **CHG-03**: Use past tense.
- **CHG-04**: Use backticks (`` ` ``) around proper names (types, fields, methods,
  packages, etc.).
- **CHG-05**: Entries should be placed in a random line position (within the
  correct section / correct release) to mitigate merge conflicts.
- **CHG-06**: Do NOT delete empty lines in the changelog — they exist to mitigate
  conflicts.

## 4. PR hygiene (informational checks)

- **PRH-01**: The PR must not be in "Ready for review" state if the build is not
  passing or if it is still WIP — should be a draft instead.
- **PRH-02**: Branch must be rebased on top of its target branch before opening
  the PR for review.
- **PRH-03**: Commits must be squashed wherever it makes sense.
- **PRH-04**: Every commit message must include the appropriate JIRA number AND
  the story title or a description of the change.

---

## 5. Methods (interface methods of each package)

- **MTH-01**: Each package interface method must be a wrapper around exactly ONE
  OpenAPI call. Combining multiple calls in a single interface method is not
  allowed. Such combined methods belong in `terraform-provider-akamai` or
  `cli-terraform`.
- **MTH-02**: Preferably ALL methods from the given OpenAPI package/family are
  covered. Missing methods should be called out.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akamai/AkamaiOPEN-edgegrid-golang](https://github.com/akamai/AkamaiOPEN-edgegrid-golang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
