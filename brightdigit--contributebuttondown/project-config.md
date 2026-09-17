---
trigger: always_on
description: This is the canonical agent instruction file for this repository. `CLAUDE.md` is a symlink to it.
---

# AGENTS.md

This is the canonical agent instruction file for this repository. `CLAUDE.md` is a symlink to it.

## Overview

ContributeButtondown is a Swift library (SPM package, product `ContributeButtondown`) that turns
Buttondown newsletter emails into Markdown files with YAML front matter, using the
[`Contribute`](https://github.com/brightdigit/Contribute) pipeline. It is consumed as a library;
there is no executable target.

It is the non-deprecated successor to `ContributeMailchimp` for brightdigit.com's newsletter
archive (brightdigit.com issue #122). Buttondown's plaintext editor stores issue bodies as
Markdown already, so this package copies the body through instead of converting HTML.

## Commands

Builds with the **Swift 6.4 toolchain** (`.swift-version` → `6.4.x-snapshot`; tools-version 6.4,
Swift 6 language mode). Use the matching snapshot / `Xcode-beta` toolchain locally.

- Build: `swift build`
- Build incl. tests: `swift build --build-tests`
- Run tests: `swift test`
- Run one suite: `swift test --filter IssueNumberingTests`

### Linting

Lint tooling is pinned via **mise** (`.mise.toml`: swift-format, SwiftLint, periphery). The entry
point is `Scripts/lint.sh`, which bootstraps tools with `mise install` then runs swift-format,
SwiftLint, and a build check. The license-header rewrite (`Scripts/header.sh`) and periphery run
locally only — the script skips them when `CI` is set.

- Full lint + autofix (local): `Scripts/lint.sh`
- Format only: `FORMAT_ONLY=1 Scripts/lint.sh`
- CI/strict mode (no autofix, fails on warnings): `LINT_MODE=STRICT CI=1 Scripts/lint.sh`

`Scripts/lint.sh` passes `-p "ContributeButtondown"` to `Scripts/header.sh`. **Never overwrite
`Scripts/lint.sh` with another package's copy** — that argument is per-package and controls the
package name written into every source header.

SwiftLint config is strict and opinionated (`explicit_acl`, `force_unwrapping`, small
`file_length` limits) — keep files small and access levels explicit.

## Architecture

Everything hangs off the `Newsletter` enum, which conforms to `Contribute`'s `ContentType` and
binds the trio of source model, front-matter translator, and markdown extractor:

- `Newsletter.Source` (`Source.swift`) — the fully-resolved issue: slug, issue number, Buttondown
  email id, archive URL, featured image URL, title, description, date, and Markdown body. Its
  `init(email:issueNo:slug:featuredImageFallback:)` builds one from a `ButtondownKit.Email`,
  throwing `ButtondownImportError.malformedArchiveURL` when `absoluteURL` will not parse, and
  substituting `featuredImageFallback` when the email carries no image.
- `Newsletter.FrontMatter` (`FrontMatter.swift`) — the **default** emitted YAML front matter. The
  field set mirrors what brightdigit.com's `NewsletterItem` reads (`issueNo`, `title`, `date`,
  `description`, `featuredImage`, `longArchiveURL`); `buttondownID` is retained for provenance
  and ignored by the site. It has a public memberwise init so consumers can construct one.
- `Newsletter.FrontMatterTranslator` — maps `Source` → `FrontMatter`, formatting the date with
  `Contribute`'s `YAML.dateFormatter`.
- `Newsletter+CustomFrontMatter.swift` — a `write(…translatedBy:)` overload taking a translator
  **instance**, so a consumer can emit any `Encodable` schema instead of `Newsletter.FrontMatter`.
  It exists because `Contribute.FrontMatterTranslator` requires `init()` and `ContentType`
  default-constructs the translator, so the typealias path cannot carry per-site configuration.
  Making the *field set itself* configurable would mean changing `Contribute`'s protocol — a Wave 0
  package — so it was deliberately not attempted.
- `Newsletter.MarkdownExtractor` — copies `source.markdown` through verbatim, stripping the
  leading `<!-- buttondown-editor-mode: plaintext -->` marker and the newlines after it.
- `IssueNumbering.swift` — how an explicit issue number is recognized in a subject. Holds the
  compiled regex; `IssueNumbering.default` is the `Issue N` / `Issue #N` form. `init(subjectPattern:)`
  **throws** rather than trapping — the pattern is consumer input, so an invalid one must not crash
  the import. Keep it that way.
- `Newsletter+IssueNumbering.swift` — issue-number assignment. `parseIssueNumber(fromSubject:numbering:)`
  pulls the number out of a subject; `assignIssueNumbers(to:continuingFrom:numbering:)` sorts
  oldest-first and fills gaps sequentially; `newIssues(from:continuingFrom:existingIssueNumbers:existingSlugs:slug:numbering:)`
  filters already-imported emails **before** numbering, which is what makes repeated imports
  idempotent. Do not reorder that filter-then-number sequence. Every `numbering:` parameter is
  defaulted, so adding one is source-compatible for callers.

Code is `#if canImport(FoundationNetworking)`-guarded for non-Apple platforms — preserve those
guards.

Strict concurrency is mandatory (Swift 6 language mode). Fix concurrency diagnostics properly
(`Sendable`, actor isolation) — never lower the language mode or silence the diagnostic.

## Tests

`Tests/ContributeButtondownTests/` uses **swift-testing** (`@Suite` / `@Test` / `#expect`), not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brightdigit/ContributeButtondown](https://github.com/brightdigit/ContributeButtondown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
