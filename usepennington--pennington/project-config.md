---
trigger: always_on
description: Content engine library targeting .NET 11 / C# 15 with union types.
---

# Pennington

Content engine library targeting .NET 11 / C# 15 with union types.

## Build & Test
- Build: `dotnet build Pennington.slnx`
- Test: `dotnet test Pennington.slnx`
- Single test: `dotnet test Pennington.slnx --filter "FullyQualifiedName~TestName"`
- Run docs site: `dotnet run --project docs/Pennington.Docs`
- CLI verbs (any host): `dotnet run --project <site>` serves; `-- build [--base-url /x] [--output dir]` generates the static site; `-- diag <info|toc|routes|warnings|translation|frontmatter|llms>` runs read-only inspection (text output) for humans and AI assistants — plus DI-discovered verbs from optional packages (`books` from Pennington.Book, `standard-site` when `PenningtonOptions.StandardSite` is configured); `-- diag --help` lists what's registered

## Project Structure
- `src/Pennington/` — Core library (Markdig, SharpYaml, AngleSharp, TextMateSharp)
- `src/Pennington.UI/` — Razor component library (TableOfContentsNav, OutlineNav, Badge, Card, CodeBlock, etc.)
  - `Pennington.UI.Styling` — components style themselves: each renders inline default class literals (in a method body, so an edit hot-reloads) for its slots, optionally selected by a component `Variant` enum (e.g. `TocVariant` Rail/Pill, whose per-slot bases live in `TocVariantStyles.For`). A nullable per-instance `*Class` param Tailwind-merges over the slot base via `ClassMerge` — a thin wrapper over the `Func<string,string,string>` delegate from `MonorailCssService.CreateClassMerger(options)` (conflicts derived from the site's own `CssFramework`). Site templates register one `ClassMerge` singleton; bare hosts leave it unregistered and the component falls back to appending (conflicting base utilities not removed). All class values must be IL string literals for MonorailCss discovery; functional classes the outline script needs (`relative`, `absolute`, `opacity-0`) stay hardcoded in markup. Pennington.UI stays MonorailCss-free (it only sees the delegate). The DocSite sidebar look is `TableOfContentsNavigation`'s `Pill` variant.
- `src/Pennington.MonorailCss/` — MonorailCSS integration (utility-first CSS generation)
- `src/Pennington.DocSite/` — Documentation site template (layout, pages, content resolver)
- `src/Pennington.BlogSite/` — Blog site template (home/archive/tag pages, blog front matter, content service)
  - Both site templates share their blog **data/logic** through core (`BlogPostQuery`, `PagedList<T>`, the `Pennington.StructuredData` JSON-LD types, and `TaxonomyContentService` for browse-by-tag). What stays template-specific is chrome: layouts, `App.razor`, the options record, and the front-matter type (`BlogPostFrontMatter` vs `BlogSiteFrontMatter` — kept separate because their `SearchOnly` policy and `Repository`/`Series` fields differ). Tag pages are `@page` components backed by a registered taxonomy axis (not `MapTaxonomy`), so they keep full chrome + search indexing.
- `src/Pennington.TreeSitter/` — Optional tree-sitter-based multi-language code-fragment extraction (`:symbol` fence, name-path addressing) via the `TreeSitter.DotNet` package
- `src/Pennington.ApiMetadata/` — Backend-neutral API-reference metadata layer: `IApiMetadataProvider` (async type/member/extension-method/xmldoc lookups keyed by uid), the `ApiTypeSummary`/`ApiTypeDetail`/`ApiMember` models, and the `IXmlDocParser`/`IXmlDocHtmlRenderer` seam. No DI entry of its own — backend packages register providers.
- `src/Pennington.ApiMetadata.Reflection/` — Reflection backend reading compiled assemblies: `AddApiMetadataFromCompiledAssembly(name, …)` registers a **keyed singleton** provider per documented library (own `MetadataLoadContext` + xmldoc index; `CompiledAssemblyApiOptions` takes assembly dirs/files).
- `src/Pennington.DocSite.Api/` — DocSite add-on publishing an API-reference content tree from a named metadata provider: `AddApiReference(name, …)` must be called **after `AddDocSite`** (it mutates `SearchIndexOptions.PrefixPriorities` and `DocSiteOptions.AdditionalRoutingAssemblies`). `ApiReferenceContentService` serves the tree (default prefix `/reference/api/`); Mdazor components under `Components/Reference/` inherit `ApiReferenceComponentBase`. Multiple libraries = repeated named registrations.
- `src/Pennington.TranslationAudit/` — Build-time translation freshness auditor (`AddTranslationAudit`): a core `IBuildAuditor` that compares git commit timestamps (libgit2) to classify each translation Up-to-date/Outdated/Missing; surfaces in the dev overlay and build report.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usepennington/pennington](https://github.com/usepennington/pennington) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
