---
trigger: always_on
description: Read these instructions before changing README.md, UPSTREAM.md, CHANGELOG.md,
---

# go-domdistiller Agent Instructions

Read these instructions before changing README.md, UPSTREAM.md, CHANGELOG.md,
benchmark claims or releases. Inspect current files and git status; preserve
unrelated edits and follow explicit user constraints.

## Library Identity

`go-domdistiller` is derived from `chromium/dom-distiller` and `kohlschutter/boilerpipe`.
Distinguish the main branch's improvements from the faithful stable branch.
Do not describe server-side extraction as computed browser layout, CSS or
JavaScript execution. Pagination is a supported independent feature.

Keep these instructions focused on `go-domdistiller`. Include other packages
for the shared benchmark comparison, not their implementation or integration
policies in this package's maintenance rules.

## Required Creator Acknowledgments

The README's License and Credits section must explicitly name:

- The Chromium Authors, creators of `chromium/dom-distiller`.
- Christian Kohlschuetter, creator of `kohlschutter/boilerpipe`, on which
	`chromium/dom-distiller` builds.
- Radhi Fadlillah, who implemented the original `go-domdistiller` port for
	Project Ratio.
- Markus Mobius, the `go-domdistiller` maintainer and MIT copyright holder;
	do not substitute the maintainer for the original port author.

Preserve the inherited licenses and notices. LICENSE-domdistiller.txt is an
empty inherited placeholder, not the complete license. Link the
[pinned Chromium notice](https://chromium.googlesource.com/chromium/dom-distiller/+/2a180397710719913340a12804affc65b789275e/LICENSE)
when presenting its BSD and Apache terms. Verify attribution against LICENSE,
LICENSE-boilerpipe.txt, NOTICE-boilerpipe.txt and the preserved upstream credits.
Do not imply endorsement by the original creators.

## Document Roles

- README.md: purpose, scope, usage and concise current quality/speed. Preserve
  useful API/examples and clearly label historical comparisons and limitations.
- UPSTREAM.md: source/dependency pins, deliberate deviations, controlled evidence,
  reproduction commands, hashes, coverage and unresolved compatibility issues.
- CHANGELOG.md: dated/versioned user-visible changes and reasons, not worker
  incidents or an audit transcript. Label documentation-only changes explicitly.
- Release notes: the same release-facing changes, benchmark definitions and
  limitations as the README/changelog, with links to detailed evidence.
- AGENTS.md: durable instructions, not current results or work-in-progress.

## Package Naming

Always identify the implementation by its full package name, including in
titles, headings, prose, tables, captions, changelogs and release notes:

- `go-domdistiller`
- `rust-domdistiller`
- `go-readabilityV2`
- `rust-readability-v2` (repository: `rust-readability`; Rust import: `rust_readability`)
- `go-trafilatura` (Go module: `github.com/markusmobius/go-trafilatura/v2`)
- `rust-trafilatura`

Never replace a package identifier with a bare algorithm name or a generic
language/algorithm label. Do not drop the language prefix or the versioned
package suffix. Give measured versions beside package names in benchmarks.
When discussing upstream projects, use their owner-qualified repository names,
not names that could be mistaken for one of these packages. Keep actual code identifiers
and import aliases unchanged; naming prose precisely is not an API rename.

## README Format

The rules below define the approved structure and required content for all six
library READMEs. Each repository's README demonstrates these rules; it is not a
substitute for this specification. Keep package-specific guidance local to its
own repository. The benchmark repository retains its methodology/results layout.

Consistency means the whole README, not just an identical benchmark table.
Use these exact level-two headings in this exact order. Every point in the
Required Content column is mandatory, not a suggestion:

| Order | Heading | Required Content |
| --- | --- | --- |
| 1 | `## Philosophy` | State all three principles in order: bring your own HTML; stay as close as possible to upstream; provide very fast native Go and Rust packages. Explain each using the required points below. |
| 2 | `## Overview` | Inputs, outputs, scope, current release and source/reference relationship. Identify which entry points fetch pages, if any. |
| 3 | `## Installation` | One command pinned to the current published version. Explain package/import names where they differ and relevant compiler requirements. Never recommend `@main`, `-u` or an unpinned branch as the default. |
| 4 | `## Usage` | One complete runnable example using local HTML, without network access or external fixtures. Show useful output. Follow with a short entry-point/result summary; link generated API docs instead of copying full type definitions. |
| 5 | `## Options` | A compact table of this package's important controls, their actual defaults and effects. Put explanations of its own behavior under level-three headings here. |
| 6 | `## Current Quality and Speed` | The shared six-package comparison with `### Extraction Speed` and `### Text Quality`. Use full package names, measured versions, common units, timing boundaries and evidence links. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markusmobius/go-domdistiller](https://github.com/markusmobius/go-domdistiller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
