---
trigger: always_on
description: Tesserae is a C# UI toolkit for building web applications, compiled to JavaScript via the **h5** compiler.
---

# CLAUDE.md

## Repository overview

Tesserae is a C# UI toolkit for building web applications, compiled to JavaScript via the **h5** compiler.

- Core UI components: `Tesserae/src/Components`
- Component factories and helpers: `Tesserae/src/Base/UI.Components.cs`
- Fluent extensions: `Tesserae/src/Extensions`
- Samples and demos: `Tesserae.Tests/`
- Project and build config: `Tesserae/Tesserae.csproj`, `Tesserae/h5.json`

## Skills

`Tesserae/skills/` is a **single Claude skill** named `tesserae`, structured per
Anthropic's skill guidance (one root `SKILL.md`, with detail layered into a
`references/` folder for progressive disclosure):

- `Tesserae/skills/SKILL.md` — the root skill. Explains the library basics, using
  components, layout with `Stack`/`Grid`, the key sizing/spacing/alignment helpers,
  and an index of every reference file and how to find it.
- `Tesserae/skills/references/<slug>.md` — one reference per component or topic
  (kebab-case slug, e.g. `button.md`, `details-list.md`, `context-menu.md`), plus
  the cross-cutting references `icomponent.md` (the `IComponent` interface and every
  fluent extension method), `core-concepts.md`, `creating-a-component.md`,
  `javascript-interop.md`, `wrap-a-javascript-library.md`, and the styling/layout
  topic docs. Each has the factory signature, key fluent methods, and an example.

These skills are written for **consumers of the Tesserae NuGet package**, not for
this repo, which is why they live under the project folder (`Tesserae/skills/`)
rather than this repo's `.claude/`.

### How the skills ship to consumers

The skill is packed into the Tesserae NuGet package and extracted into a
referencing project's `.claude/skills/tesserae/` on build. The plumbing lives in
[`Tesserae/Tesserae.csproj`](Tesserae/Tesserae.csproj) and
[`Tesserae/buildTransitive/Tesserae.targets`](Tesserae/buildTransitive/Tesserae.targets):

- The csproj packs everything under `Tesserae/skills/**` (the root `SKILL.md` and
  the whole `references/` tree) into the package's `skills/` folder, and a
  `_WriteSkillsVersion` target stamps the package version into
  `skills/.skills-version` (`NoDefaultExcludes` lets the dot-file pack).
- `Tesserae.targets` (auto-imported via `buildTransitive/`, so it reaches both
  direct and transitive consumers) walks up from the consuming project to find a
  `.claude` folder. If one exists, it compares the shipped `.skills-version`
  against the installed marker and, when they differ, wipes and re-copies the
  whole `skills/` payload into `.claude/skills/tesserae/`. No `.claude` folder →
  it does nothing.

So a Tesserae app that has a `.claude` folder automatically gets the skill
refreshed whenever it upgrades the package. Changes here reach consumers on the
next package version bump (the version marker is what triggers the re-copy). Note
the install folder is `tesserae` (the skill `name`, lowercase kebab-case); the
targets *filename* must stay `Tesserae.targets` (= the csproj `<PackageId>`) so
NuGet auto-imports it, and `_SkillsPackageId` inside the targets sets the install
folder name.

### Keep skills in sync with the code

The skill is documentation that drifts out of date if the code changes underneath
it. Whenever you change the public surface of the toolkit, update the skill in the
same change:

- **New component** — add `Tesserae/skills/references/<slug>.md` (slug = the
  doc/kebab-case name), link it from related references, and add it to the index in
  `Tesserae/skills/SKILL.md`.
- **Changed factory or fluent method** (renamed, new/removed parameters, new
  configuration method, changed default) — update that component's reference so the
  signatures and examples still compile.
- **New or changed `IComponent` extension method** (under
  `Tesserae/src/Extensions/`) — update `references/icomponent.md` (and the sizing
  cheat-sheet in `SKILL.md` if it's a common one).
- **Removed component** — delete its `references/<slug>.md`, drop it from the
  `SKILL.md` index, and fix any "Related" links that pointed at it.

The root `SKILL.md` `name` must be `tesserae` and its `description` must state what
it does and when to use it (no `<`/`>`). Keep `SKILL.md` a focused overview and
push detail into `references/`. The same applies to the matching pages in the
`documentation` repo under `tesserae/` — update them alongside the references.

## Installing h5

Install or update the h5 compiler and the dotnet serve tool globally before getting started:

```bash
dotnet tool update --global h5-compiler
dotnet tool update --global dotnet-serve
## Build

```bash
dotnet build
```

The h5 compiler translates C# to JavaScript. Output lands in `bin/Debug/netstandard2.0/h5/` (or `bin/Release/...`).

To serve locally:

```bash
cd bin/Debug/netstandard2.0/h5/
dotnet serve --port 5000
```

## UI composition patterns

- Component creation goes through the static `UI` class (`UI.Components.cs`), which exposes factory methods like `UI.Button`, `UI.TextBlock`, etc.
- `UI` is a static partial class with a static constructor used as the central entry point.
- Components are configured via fluent-style extension methods (e.g., `UI.Id`, `UI.Class`, `UI.Do`).

## Conventions

### Type safety


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [curiosity-ai/tesserae](https://github.com/curiosity-ai/tesserae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
