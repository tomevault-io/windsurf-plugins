---
trigger: always_on
description: <!-- Repository-level Copilot instructions for Serenity platform -->
---

<!-- Repository-level Copilot instructions for Serenity platform -->
# Copilot instructions — Serenity (repo root)

Key points (quick):
- Root build is driven by MSBuild/Visual Studio (`Serenity.slnx`) for .NET projects and `pnpm`/`node` for TypeScript packages under `packages/` and `src/typescript`.
- Many packages (e.g., `packages/corelib`, `packages/sleekgrid`) contain their own `README.md` and a `.github/copilot-instructions.md` with package-specific details — check those for specialized rules.
- Use project scripts (`pnpm -r build`, `pnpm -r tsc`, `pnpm -r test`) from the repository root where appropriate; .NET builds are via `build\build.csproj` or opening `Serenity.slnx` in Visual Studio.

Repository architecture (concise):
- .NET backend: projects live under `src/` (e.g., `src/core/Serenity.Net.Core.csproj`, `src/web/Serenity.Net.Web.csproj`) and are composed into `Serenity.slnx`. These projects provide services, code-generation hooks, and MSBuild targets that integrate TypeScript builds.
- TypeScript frontend and libraries: under `packages/` and `serene/` (app template). Notable package: `packages/corelib` — core TypeScript utilities, event system, UI widgets, and runtime registration conventions.
- Common-features: reusable .NET features and demo projects in `common-features/`.
- Build/release orchestration: top-level `build/` contains MSBuild helpers; `build.cmd` invokes `dotnet run --project build\build.csproj`.

Critical developer workflows (explicit commands)
- Full repo (Windows/pwsh):
  - Install Node deps (root):
    ```powershell
    pnpm install
    ```
  - Build TypeScript packages and .NET projects (recommended):
    ```powershell
    # from repo root
    dotnet run --project .\build\build.csproj --
    # or use the simpler wrapper (Windows)
    .\build.cmd
    ```

    Note that `build` does not do type checking; use `tsc` for that. `pnpm -r dts` uses typescript to generate bundled declaration files so running it is recommended when changing public APIs and it also covers type checking. `test` also runs `build` first.

  - Run an individual .NET project (example web):
    ```powershell
    dotnet run --project .\serene\src\Serene.Web\Serene.Web.csproj
    ```

  - Run script tests for individual package from the package folder
    ```powershell
    pnpm test
    ```

- Don't use ` -- ` to pass args to any of the pnpm commands, e.g. use `pnpm test -t MyTest` instead of `pnpm test -- -t MyTest`, use `pnpm test --coverage` instead of `pnpm test -- --coverage`.
- Check current directory with `pwd` if unsure where you are in the repo or an individual package.
- Assume working in a windows environment unless otherwise specified, so tools like grep, sed, awk may not be available. Powershell equivalents are preferred.

Important conventions and patterns (repo-specific)
- Global registration & runtime metadata: The TypeScript corelib relies on symbol-based registration for enums, types and script data. See `packages/corelib/src/base/system.ts` and `scriptdata.ts` (package `@serenity-is/corelib`) for examples. When adding runtime metadata follow existing register/unregister helpers.
- Event and UI patterns: UI components and editors use a custom event system (look in `packages/corelib/src/base/` for event utilities and `src/ui/` for editor patterns). Prefer reusing existing factories (dialogs, editors) rather than creating ad-hoc DOM manipulation.
- .NET ↔ TypeScript integration: MSBuild targets call TypeScript build steps via `RunTSBuild` and related targets in `src/Directory.Build.targets` and `build/Package.Build.props`. Changing package outputs often requires updating the corresponding `.csproj` or MSBuild props.
- Packaging: Some packages are consumed via NuGet wrappers that place JS under `node_modules/.dotnet/<package>`; check `packages/*/README.md` which documents the hybrid NuGet/NPM usage.
- Data binding patterns: Use lookup system for dropdowns/editors (`LookupEditor` in `packages/corelib/src/ui/editors/lookupeditor.ts`). Script data system handles dynamic content loading via `~/DynamicData/` endpoints.
- Testing: Vitest workspace configured in `vitest.workspace.json` runs tests across packages. Use `pnpm -r test` from root.

Examples to copy patterns from
- Adding a new editor widget: model after `packages/corelib/src/ui/editors/lookupeditor.ts` and `src/ui/editors/*.tsx` — the pattern uses `ScriptData` binding and global registration.
- Runtime metadata registration: use `registerEnum(myEnum, "MyEnum")` or `registerClass(MyClass, "MyClass")` from `packages/corelib/src/base/system.ts`.
- MSBuild integration: examine `src/Directory.Build.targets` and `build/Package.Build.props` for targets that run TS build and package artifacts.
- Event handling: see fluent-events in `packages/corelib/src/base/fluent-events.ts` for the custom event system.

Where to look first (high-signal files)
- `Serenity.slnx` — overall project composition and which projects are built together.
- `build/build.csproj` and `build.cmd` — orchestrates higher-level build steps for the repo.
- `packages/corelib/.github/copilot-instructions.md` and `packages/corelib/README.md` — package-specific agent guidance and examples.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serenity-is/Serenity](https://github.com/serenity-is/Serenity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
