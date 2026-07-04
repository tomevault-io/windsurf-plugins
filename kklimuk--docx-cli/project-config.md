---
trigger: always_on
description: CLI for AI agents to read, edit, and comment on `.docx` files. JSON-AST output, locator-based addressing, full format fidelity via in-place XML mutation.
---

# docx-cli

CLI for AI agents to read, edit, and comment on `.docx` files. JSON-AST output, locator-based addressing, full format fidelity via in-place XML mutation.

**Bun, not Node.** Use `Bun.file`, `Bun.write`, `Bun.env`, `Bun.$`. Bun loads `.env` automatically — no dotenv.

Subsystem-specific guidance lives in nested CLAUDE.md files that load when you edit those folders. If you need to add a new CLAUDE.md to describe a new practice for a part of the system, do so.

## Conventions

These conventions are NOT SUGGESTIONS. These are rules.

- **All stdout goes through `respond()` (JSON ack) or `writeStdout()` (text)** from `src/cli/respond.ts` — never `process.stdout.write`. Both use `Bun.write(Bun.stdout, ...)`; the 64 KB truncation that bites on early exit is real and silent, and these helpers are the only safe path.
- **File naming**: kebab-case, named after the primary export (`xml-node.ts` → `XmlNode`).
- **Newspaper ordering.** The entry point (primary export) goes at the top; its dependencies follow in the order it uses them, then _their_ dependencies, and so on — a file reads top-to-bottom like a newspaper. Use hoisted `function` declarations for internal helpers so this works at runtime; arrow functions only for inline callbacks and short utilities. Types are usually not the primary exports and should go below the functions/classes that are.
- **Feature nesting** When a file accumulates too many dependencies to be read well with newspaper ordering (> 300 lines), split them into a separate folder/file named after the feature they're working on. It should be a folder if it is going to represent a logical feature of dependencies. This nesting can continue indefinitely if subfeatures have subfeatures of their own.
- **JSX is for emitters only.** Files that construct fresh XML can be `.tsx`; readers/locators/analysis stay `.ts`. Components are PascalCase, accept props, may return `NullableXmlNode` (null skipped by flatten). Attribute names with colons use the hyphen shortcut (`w-val="x"` → `w:val="x"`) or JSX spread.
- **Component vs view vs lens vs free function**: four shapes, one decision tree.
  - A pure `props → XmlNode` builder is a PascalCase **component** — destructure its props in the signature (no `props.x` access), and don't take a `Document` (or any package state).
  - Stateful OOXML state lives in **tree-owning views**, embedded as fields on `Document`: `Body`, plus one view per OPC part — `StylesView`, `NumberingView`, `CommentsView`, `NotesView`, `RelationshipsView`, `ContentTypesView`, `SettingsView`, `CorePropertiesView`, `MarginalsView`. Each owns its part's `XmlNode` tree and any maps keyed to it, and exposes a `fromPackage`/`fromXml`/`writeTo` lifecycle (`register` too, for the lazily-provisioned ones). `MarginalsView` is the one that owns MANY parts (every `word/header{N}.xml` / `word/footer{N}.xml`) keyed by part name rather than one — so it has no single `register`; the `Marginals` lens mints each part's rel + content-type as it allocates it. Cross-view dependencies (e.g., `NotesView.ensureNoteStyles(stylesView)`) are passed as method arguments — no view reaches up to `Document`.
  - **Cross-cutting lenses** (`Images`, `Hyperlinks`, `Equations`, `TrackChanges`, `Comments`, `Fonts`, `Marginals`) are NOT fields on `Document` — they're stateless, constructed at the call site: `new Images(document).add(source)`, `new TrackChanges(document).accept(["tc0"])`, `new Marginals(document).set(sectPrs, "footer", "default", spec)`, `await new Fonts(document).setDefault("Times New Roman")`. They hold only a back-reference; the embedded views are the state they reach through. `Marginals` is the header/footer authoring lens — the noun pair `docx headers`/`docx footers` share it via `MarginalKind` the way `footnotes`/`endnotes` share `Note` — reaching through `MarginalsView` (part trees) + relationships/content-types (part registration) + settings (the even/odd toggle) + the live `<w:sectPr>` reference nodes (see [src/core/marginals](src/core/marginals/CLAUDE.md)). `Fonts` is the one that also touches an UNMODELED part — the document font lives in BOTH `word/styles.xml` `<w:docDefaults>` (owned by `StylesView`) and `word/theme/theme1.xml`'s `<a:fontScheme>` (not a view: read/mutated/staged through `Pkg` only when `set-default-font` runs, so unrelated saves never re-serialize the theme blob).
  - **Free functions** are reserved for: pure builders (the components above), the AST reader (`src/core/ast/read.ts` — Document's construction pass; populates the embedded views from XML and is the sole assigner of `tcN` ids), and emitter helpers in `src/core/blocks`/`table`/`sections` that thread a `Document` because they touch many slices in one call. If a free function's body operates on one slice of `document`, make it a method on that slice's view instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kklimuk/docx-cli](https://github.com/kklimuk/docx-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
