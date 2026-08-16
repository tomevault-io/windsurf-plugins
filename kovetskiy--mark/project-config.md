---
trigger: always_on
description: Guidance for AI coding agents (and new human contributors) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (and new human contributors) working in this repository.

`mark` reads Markdown and writes **Confluence storage format** — an XML dialect, not
HTML. Most of the subtle bugs in this project's history come from forgetting that. Read
the Invariants section before changing anything under `renderer/`, `transformer/`,
`parser/`, `macro/`, `includes/`, or `stdlib/`.

## Build, test, lint

```shell
make build                 # -> ./mark  (CGO_ENABLED=0, ./cmd/mark)
make test                  # go test -race -coverprofile=profile.cov ./... -v
go test ./... -count=1     # faster loop
golangci-lint run ./...    # config in .golangci.yml; CI runs this
markdownlint-cli2          # config in .markdownlint-cli2.jsonc; CI runs this on *.md
```

Go 1.26. The module path is `github.com/kovetskiy/mark/v16` — the major version is part
of the path, so a major bump means rewriting every internal import.

`d2` and `mermaid` tests launch headless Chrome and take ~4s each; a full `go test ./...`
is dominated by them. There is currently no `-short` skip.

## Layout

| path | role |
| --- | --- |
| `cmd/mark` | thin `main`; flag wiring lives in `util/` |
| `util/` | CLI flags, config file/env sourcing, credential resolution |
| `mark.go` | orchestration: `Run` (glob → loop) and `ProcessFile` (the whole per-file pipeline) |
| `metadata/` | `<!-- Header: -->` comments and YAML front matter → `Meta` |
| `page/` | ancestry/folder resolution, relative-link rewriting, relocation |
| `confluence/` | REST client (v1 `/rest/api` + v2 `/api/v2`), page cache |
| `attachment/` | checksum, upload/update, link rewriting |
| `markdown/` | goldmark assembly; `CompileMarkdown` is the entry point |
| `parser/` | goldmark inline/block parsers (`<ac:*/>` tags, mentions, dates) |
| `transformer/` | goldmark AST transformers (macros, includes, GH alerts, details, layout, `<img>`) |
| `renderer/` | goldmark node renderers → storage format |
| `stdlib/` | the `text/template` set that emits all `<ac:*>` markup |

Pipeline in `ProcessFile`: read → normalise CRLF → extract metadata → resolve relative
links → resolve/create page + ancestry → resolve attachments → `CompileMarkdown` →
resolve inline attachments → wrap in `ac:layout` → optionally merge inline comments →
update page → sync labels.

## Invariants

**1. Output must be well-formed XML.** Confluence rejects the whole page with
`BadRequestException` if it is not. Unbalanced tags, stray `&`, or an unescaped quote in
an attribute break the entire upload, not just one element.

**2. Escape through the `stdlib` template funcs, never by hand.**

- `xmlesc` — any interpolated text or attribute value.
- `cdata` — any text placed inside `<![CDATA[...]]>`; it splits an embedded `]]>` across
  two CDATA sections, which is the only legal way to escape it.
- `convertAttachment` — values destined for `ri:filename`; slash-flattens *and* escapes.

Titles, filenames, and `ri:content-title` have each caused a malformed-XML bug in the
past. If you add a template that interpolates user-controlled text, pipe it through one
of these.

**3. Macros and includes are expanded *before* goldmark parses.** Macro bodies routinely
contain raw `<ac:...>` XML that goldmark would escape or mangle — particularly inside
table cells. `macro.ExtractMacros` and `includes.ProcessIncludes` run on the raw bytes in
`CompileMarkdown` before the converter is built. Do not move this work into an AST
transformer without understanding why it was moved out of one.

**4. goldmark priorities run in opposite directions for parsers and renderers.**

- *Node renderers*: registered in ascending priority order, so a **larger** number is
  registered later and **overrides** an earlier registration for the same node kind. The
  GH-alerts blockquote/text renderers use `200` specifically to beat the `100` defaults.
- *Inline/block parsers*: a **smaller** number is tried **first**. `ConfluenceTagParser`
  uses `199` to run before goldmark's own link parser at `200`, so that `<ac:*/>` tags are
  not parsed as links.

Getting this backwards silently produces the default rendering.

**5. AST transformers cannot return errors.** goldmark's `ASTTransformer` interface has no
error return, so `transformer.PipelineTransformer` accumulates errors internally and
`CompileMarkdown` retrieves them with `Pipeline.GetError()` after `Convert`. A transformer
that fails silently without recording into the pipeline will produce a wrong page and a
zero exit code.

**6. Two compile paths must both keep working.** `CompileMarkdown` (default, GH alerts +
pipeline transformer) and `CompileMarkdownLegacy` (the original renderer set). They are
compared against each other in `markdown/transformer_comparison_test.go`.

**7. Attachment checksums are content-addressed, but diagrams are source-addressed.**
Checksums live in the remote attachment's comment behind the `AttachmentChecksumPrefix`
(`mark:checksum:` followed by a space).
Mermaid and d2 attachments set `Checksum` from the *diagram source*, not the rendered PNG
bytes, because Chrome's output is not byte-stable across environments. `ResolveAttachments`
skips checksum computation when `Checksum` is already set — preserve that.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kovetskiy/mark](https://github.com/kovetskiy/mark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
