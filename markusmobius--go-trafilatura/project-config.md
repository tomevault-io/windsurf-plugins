---
trigger: always_on
description: These are instructions for LLM agents and maintainers updating README.md,
---

# go-trafilatura Agent Instructions

These are instructions for LLM agents and maintainers updating README.md,
UPSTREAM.md, CHANGELOG.md, benchmark claims and releases. Read current files
and git status first; preserve unrelated work and explicit user constraints.

## Library Identity

This is a supplied-HTML extraction library, not an application-worker project.
Use the approved nine-section format below, with package-specific APIs and
limits. `go-trafilatura` follows the pinned `adbar/trafilatura` non-fallback core with documented
differences. Compatibility tests are evidence, not proof of complete parity.

FAST disables external fallback, not native recall or baseline recovery.
Non-FAST uses only internally generated bundled readability-lxml. Standalone
extractors are outside this package's API. Keep application-worker history and
other packages' integration policies out of its maintenance rules.

## Required Creator Acknowledgments

The README's License and Credits section must explicitly name:

- Adrien Barbaresi, creator of the original `adbar/trafilatura` Python package;
	link the upstream project and his 2021 ACL/IJCNLP paper.
- Radhi Fadlillah, author of the initial `go-trafilatura` port from the Python
	`adbar/trafilatura` package; explicitly distinguish initial port authorship
	from current maintenance.
- Markus Mobius, maintainer of `go-trafilatura`; do not attribute the original
	Python package or initial Go port to the current maintainer.
- Arc90 for the original algorithm, starrhorne and iterationlabs for the Ruby
	port, and gfxmonk for the Python port, as named in `adbar/trafilatura`'s
	bundled readability-lxml header. Retain its links to the
	`timbertson/python-readability` and `buriy/python-readability` contributors;
	do not turn the two Ruby authors into an invented repository name.

Preserve the Apache-2.0 LICENSE and original notices in adapted source files.
Check UPSTREAM.md and the pinned upstream source before changing attribution.
Name the creators in prose, not just repository links; do not imply endorsement.

## Candidate Removal Explanation

Explain that candidates extracted before `go-trafilatura` input cleanup can retain
long boilerplate that passes length checks and replaces article content. Do not
say `go-domdistiller` performs no cleaning: the relevant difference is input
preparation. Use the historical same-version control, clearly labeled: `go-trafilatura` 2.2.2
selected external fallback on 780/6,554 pages (11.90%) with generated candidates
and 2,408/6,554 (36.74%) with supplied candidates; `go-domdistiller` selections were
4 versus 1,614. The complete 2.2.6 lxml-only policy measured 202/6,554 (3.08%).
That last reduction also changes the algorithm set and is not solely the effect
of candidate removal. Selection rates are not accuracy scores. Count final
returned sources, not calls or temporary selections; keep failures in the
denominator and internal recovery separate. See the controlled evidence in
UPSTREAM and the benchmark repository before adding or changing numbers.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markusmobius/go-trafilatura](https://github.com/markusmobius/go-trafilatura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
