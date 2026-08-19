---
trigger: always_on
description: A compiler for structured Markdown documents. It parses and validates Markdown
---

# docc

A compiler for structured Markdown documents. It parses and validates Markdown
and YAML frontmatter against a schema, then emits self-contained `.docx` through
the configured theme.

See `README.md` for the schema format and CLI usage. `docs/philosophy.md` is
the scope document: identity, non-goals, the closed content model, and the
way-forward list. Read it before adding a feature — most proposals it rejects
were once accepted here and later removed.

## Layout

```
cmd/docc/            CLI: check | read | build | init | profile | doctor | lsp |
                     types | describe | example | themes | explain | version
internal/diag/       Diagnostic type, source-excerpt and JSON rendering
internal/parse/      goldmark wrapper: frontmatter split, block tree, positions
internal/schema/     doc-type spec loading and `extends` resolution
internal/sema/       validation passes → diagnostics
internal/ir/         parsed document → render tree
internal/emit/       render tree + schema + theme → docx parts, emit.Validate
internal/theme/      theme loading and `extends` resolution
internal/project/    .docc directory discovery
internal/profile/    pack manifests, resolution order, Git install, provenance
internal/defaultpack/ the starter pack embedded in the binary (`builtin` source)
internal/starter/    `docc init`: copies the default pack out as a checkout
internal/lsp/        editor diagnostics over stdio
internal/docx/       .docx writer — stdlib only, no template, deterministic
testdata/            golden corpus: schemas/, good/, bad/, golden/
```

Scope guard: docc contains nothing AgentSkill- or agent-host-related. Skills
are built by the pack repositories that own the drafting knowledge; docc is
the compiler they invoke. Do not reintroduce packaging, plugin manifests or
skill prose here.

## Conventions

- `task` runs the full CI chain. Run it before committing.
- Formatting is `gofumpt`, enforced by the pre-commit hook (`task hooks:install`).
- Dependencies stay minimal: `goldmark` for markdown, `goccy/go-yaml` for YAML
  positions. Do not add a dependency for something the stdlib does.

## Working on the checker

- **Every diagnostic needs a source position and a hint.** A message that says
  what is wrong but not what to do is incomplete. Anchor on a line that actually
  relates to the problem — a caret under an unrelated line is worse than a
  file-level diagnostic.
- **Diagnostic codes are stable.** Never renumber a released code; schemas,
  `docc explain` and agent workflows reference them.
- **Passes collect, they do not stop.** All checks run and append to one list.
  An author fixing one error at a time runs the compiler ten times.
- **Adding a check:** implement it in `internal/sema/rules.go`, register it in
  `registry`, document it in the README table. Schemas select checks by name and
  supply their own code and severity.
- **Schema/theme mismatches are caught in `emit.Validate`, not at load.** It is
  the only place holding both. Every style a schema maps, every field a theme
  interpolates and every definition a render rule names is checked there, before
  a single paragraph is built.
- **Adding a diagnostic code:** add it to `explanations` in `cmd/docc/main.go`.

## Inheritance

Schemas and themes both have `extends:`, and they must keep behaving the same
way. Two different inheritance semantics in one product is a defect.

- **Maps merge key-wise; ordered sequences are replaced wholesale** when the
  child declares them. Half a letterhead is worse than a restated one.
- **A theme merges YAML documents, then decodes once** (`internal/theme`),
  rather than merging decoded structs. Only the document distinguishes a key
  left out from a key set to its zero value, and both cases are real: a child
  omitting `formats:` must keep the parent's month names, and a child setting
  `title_page: false` under a parent that set it true must win. Adding a field
  to `Theme` therefore needs no merge code — keep it that way.
- **A name beginning with `_` is a fragment**: extended, never selected.
  `Set.Names` hides them and `Set.Get` rejects them, so a schema cannot name one.
- **Resolution stays inside one directory.** No cross-pack parent. A base pack
  silently changing header spacing in every firm that installed it is the
  failure a compiler exists to prevent.

## The embedded default pack

`internal/defaultpack/files` is a real profile pack — manifest, schemas,
themes — compiled into the binary. `Resolve` falls back to it as the `builtin`
source, so resolution always answers and an unconfigured docc works out of the
box. Two invariants: the embedded pack must always validate and build
(`TestStarterProfile` in `internal/emit` guards it), and the fallback must
never mask a legacy `.docc/schemas` layout — that path fails loudly instead,
because silently compiling against schemas nobody chose is the failure a
compiler exists to prevent. The pack materializes into the XDG store
content-addressed by its hash; never resolve it from a path inside the
repository.

## Testing

`testdata/` is the regression suite, checked against `testdata/schemas/`:

- `good/` — must produce zero errors (`TestGoodDocumentsHaveNoErrors`) *and* build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinzehnder/docc](https://github.com/kevinzehnder/docc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
