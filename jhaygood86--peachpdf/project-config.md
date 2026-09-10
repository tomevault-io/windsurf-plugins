---
trigger: always_on
description: validates a JSON entry's `propertyPath`/`csharpDataType` against the *real* `CssBox`/`SvgElement` shape at
---

# PeachPDF

Pure .NET HTML → PDF rendering library. No external process dependency (no Puppeteer/wkhtmltopdf/gs) — HTML parsing, CSS cascade, layout, and PDF writing all run in-process. Targets .NET 8 and .NET 10.

## Documentation map

Read these before making non-trivial changes in their area — they are the source of truth, not this file:

- [docs/index.html](docs/index.html) — doc site marketing-style landing page (hero, feature cards, guide cards)
- [docs/why-peachpdf.html](docs/why-peachpdf.html) — "Why PeachPDF?" marketing page: HTML/CSS-as-the-API pitch, free/open-source pitch, comparison table vs other .NET PDF libraries (IronPDF, PuppeteerSharp, DinkToPdf, wkhtmltopdf, Prince, QuestPDF, PDFsharp/MigraDoc, Aspose)
- [docs/showcase.html](docs/showcase.html) — feature showcase gallery; cards (thumbnail + PDF + HTML-source links) are driven by `site.data.showcases`, which pages.yml generates at site build time by running the TestHarness (`docs/showcase/` and `docs/_data/showcases.json` are gitignored build output, never source)
- [docs/getting-started.md](docs/getting-started.md) — install, quick start, thread safety / guide index
- [docs/architecture.md](docs/architecture.md) — how HTML becomes a PDF: parser, DOM, CSS, layout, painting, PDF renderer
- [docs/testing.md](docs/testing.md) — reader-facing overview of how the project is tested: the xUnit suite, the CI matrix, the 90% diff-coverage gate, and two-renderer (PDFium + MuPDF) rasterization verification (contributor-facing commands/conventions live in `CONTRIBUTING.md`)
- [docs/html-css-support.md](docs/html-css-support.md) — full HTML/CSS compatibility matrix (elements, properties, selectors, at-rules, gaps, extensions, PDF metadata extraction, tagged PDF)
- [docs/supported-svg-features.md](docs/supported-svg-features.md) — full SVG compatibility matrix (inline `<svg>` and standalone), rendered as real vector PDF content
- [docs/usage-examples.md](docs/usage-examples.md) — copy-pasteable API usage (local HTML, MHTML, HTTP fetch, thread safety, fonts, enabling tagged PDF, ASP.NET Core/Azure Functions)
- [docs/cli.md](docs/cli.md) — the standalone `peachpdf` command-line tool: install (per-platform Native AOT binaries), usage, and the full argument reference
- [docs/support.md](docs/support.md) — free (GitHub issues) and paid (Peach State Technologies) support options
- [docs/sponsorship.md](docs/sponsorship.md) — GitHub Sponsors info; sponsors get paid support under the same terms as customers
- [docs/license.md](docs/license.md) — BSD 3-Clause license text, third-party component licenses, license FAQ
- [README.md](README.md) — package overview, install, quick start, fonts

Not documentation, but read alongside it — the internal dev notes, one file per entry:
[.claude/accepted-gaps/](.claude/accepted-gaps/) (limitations already argued through; don't relitigate
one without new information), [.claude/recent-fixes/](.claude/recent-fixes/) (the reasoning and
traps behind each recent change), [.claude/migration-notes/](.claude/migration-notes/) (user-visible
behavior changes since the last release, awaiting a release notes pass) and
[.claude/invariants/](.claude/invariants/) (what a future change must not break or re-derive). See
[Out of scope / accepted gaps](#out-of-scope--accepted-gaps-dont-relitigate-without-new-information),
[Recent fixes](#recent-fixes), [Migration notes](#migration-notes) and
[Invariants and traps](#invariants-and-traps) below.

When you add or change user-facing features, update the relevant doc page (and its `README.md`/`docs/getting-started.md` cross-links) in the same change, rather than as a follow-up — this repo's convention (established by the SVG 1.0 coverage work) is docs land with the feature.

User-facing documentation (`docs/**` and `README.md`) must be **free-standing**: it may cross-link other doc pages, [MDN](https://developer.mozilla.org), or — only where MDN has no suitable page — the upstream specification on its official URL (`w3.org`/`whatwg.org`). Do **not** cite GitHub issues or PRs in documentation (no `#123` references, no `github.com/.../issues/...` links) — an issue number is not durable, reader-facing reference material. (This applies to the docs and README only; the internal dev notes — `.claude/recent-fixes/**` and `.claude/accepted-gaps/**` — may still reference issues, since they are engineering history, not documentation. `docs/support.md` linking the issue **tracker** as a support channel is also fine — that is a support instruction, not a behavior citation.)

When you open a pull request that fixes a tracked GitHub issue, reference that issue in the PR **description** with a closing keyword (`Fixes #123` / `Closes #123`, one per issue the PR resolves) so GitHub links and auto-closes it on merge. The PR description is exactly where issue references belong — it is engineering/review context, not reader-facing documentation, so this is the complement to the docs rule above, not an exception to it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhaygood86/PeachPDF](https://github.com/jhaygood86/PeachPDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
