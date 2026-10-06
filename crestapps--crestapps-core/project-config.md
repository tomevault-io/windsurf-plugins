---
trigger: always_on
description: Use these instructions when working in this repository.
---

# CrestApps.Core Development Instructions

Use these instructions when working in this repository.

## Project overview

`CrestApps.Core` is the standalone framework repository for CrestApps shared libraries. It contains reusable .NET packages for AI, orchestration, chat, templating, document processing, SignalR, storage, and sample hosts.

- **Target framework:** .NET 10
- **Docs site:** `src\CrestApps.Core.Docs`
- **Sample hosts:** `src\Startup\CrestApps.Core.Mvc.Web`, `src\Startup\CrestApps.Core.Aspire.AppHost`
- **Tests:** `tests\CrestApps.Core.Tests`

Orchard Core-specific implementation details belong to the separate site at <https://orchardcore.crestapps.com>.

## Build and test

Use the repository root unless a command says otherwise.

```bash
dotnet build .\CrestApps.Core.slnx -c Release /p:NuGetAudit=false
dotnet test .\tests\CrestApps.Core.Tests\CrestApps.Core.Tests.csproj -c Release /p:NuGetAudit=false
```

For root asset tooling:

```bash
npm install
npm run rebuild
```

For the docs site:

```bash
cd src\CrestApps.Core.Docs
npm install
npm run build
```

## Repository layout

Important folders:

- `src\Abstractions` - public contracts
- `src\Primitives` - concrete framework features and providers
- `src\Stores` - persistence implementations
- `src\Utilities` - shared helpers
- `src\Startup` - runnable sample hosts
- `src\CrestApps.Core.Docs` - Docusaurus docs
- `tests\CrestApps.Core.Tests` - unit tests

## Documentation expectations

When a change affects public behavior, configuration, setup, or project guidance:

1. Update the relevant page under `src\CrestApps.Core.Docs\docs`
2. Update the changelog for the active in-development release under `src\CrestApps.Core.Docs\docs\changelog`
3. Build the docs site

### Changelog conventions

- Changelog files are named after their version with no `v` prefix (for example `1.1.0.md`, `2.0.0.md`).
- The next planned release is **2.0.0**. Document all upcoming changes — new features, fixes, dependency upgrades, branching or workflow changes, and other notable repository-level updates — in `src\CrestApps.Core.Docs\docs\changelog\2.0.0.md` until it ships.
- When a new development cycle begins, add a changelog file for that version (again with no `v` prefix), register it in `src\CrestApps.Core.Docs\sidebars.js` and `src\CrestApps.Core.Docs\docs\changelog\index.md`, and document ongoing work there.

Keep the docs focused on `CrestApps.Core`. If you need to mention the Orchard Core implementation, treat it as a related downstream product and link to <https://orchardcore.crestapps.com>.

## Coding guidance

- Follow `.editorconfig`
- Prefer constructor injection
- Do not add `ArgumentNullException.ThrowIf...` guards in constructors
- Add null guards in public implementation methods when a non-nullable input is required and the method does not intentionally support `null`
- Skip null guards for nullable or intentionally null-tolerant parameters
- After the last null-check/argument-validation line in a method, add a blank line before the next statement
- If a method has only one null-check/argument-validation line, still add a blank line after that final guard line
- Add a blank line before a `return` statement unless the `return` is the first statement inside a `{ ... }` block
- Never use more than one consecutive blank line
- Add a blank line before and after `if` blocks, `switch` statements, and loops unless the block is immediately preceded by `{`
- Do not add a blank line between an `if`/`else`/`switch`/loop condition and its opening `{`
- Do not add a blank line immediately after `#pragma warning disable` or immediately before `#pragma warning restore`
- Do not add a blank line immediately after `#pragma warning restore` when the next line is `{`
- Add a blank line before a `#pragma warning disable` block when it starts a new member after a closing `}`
- Add a blank line between a `#pragma warning restore` and the next `#pragma warning disable` when they guard separate members
- Format conditional operators across multiple lines with the condition on its own line and the `?` and `:` tokens on their own indented lines
- Use `var` consistently with repository style
- Do not use `global using` files; add explicit `using` directives at the top of each file instead.
- Prefer top-of-file `using` directives over fully qualified type names in code.
- Only use expression-bodied members when the entire member fits on a single short line; use a full block body for anything longer or split across lines
- Avoid `DateTime.UtcNow`; prefer injected `TimeProvider`.
- Keep public docs and comments honest to the current code.
- Always document every method, including constructor overloads, with accurate XML `<summary>` and `<param>` blocks for every argument.
- Only add XML `<param>` tags for parameters that actually exist on the documented member, and keep them in the exact same order as the signature.
- Always document publicly accessible properties with accurate XML `<summary>` blocks.
- Always insert a blank line before XML `<summary>` documentation blocks unless they are immediately preceded by `{`.
- Never insert a blank line between an XML documentation block and the member it documents.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrestApps/CrestApps.Core](https://github.com/CrestApps/CrestApps.Core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
