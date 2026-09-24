---
trigger: always_on
description: Guidance for Claude Code working in this repository. Only what you can't infer from the code —
---

# CLAUDE.md

Guidance for Claude Code working in this repository. Only what you can't infer from the code —
architecture is documented in [docs/architecture.md](docs/architecture.md) and the rest of `docs/`.

## What it is

A VS 2022/2026 extension (VSIX, C#/.NET Framework 4.8, `Corsinvest.VisualStudio.Agents`) that hosts
two pane types plus an in-process MCP server exposing the IDE to the CLI. The chat UI is a WebView2
app in TypeScript + Lit.

It **drives** the real `claude.exe` (npm `@anthropic-ai/claude-code`) — never bundled (licensing),
never forked. Version differences are handled by feature-detection, not by pinning a CLI version.

## The IDE is right there

When this solution is open in a Visual Studio running the extension, the `mcp__vs__*` tools reach
that IDE — the one that has built this code and holds its semantic model. Ask it rather than
shelling out or re-reading files: `ide_get_diagnostics` for what the compiler thinks,
`nav_find_references` for callers, `ide_read_output` for what a build or a debug session printed.
The extension is its own best test case, so the IDE you are talking to is usually running the
build you just made.

## Build

Use `mcp__vs__build_solution` when that IDE is open: it drives it, so there is no MSBuild path to
resolve and no clash with a debug session holding the assembly, and the errors come back as
file/line/message. Otherwise:

```powershell
msbuild cv4vs-agents.slnx /t:Build /p:Configuration=Debug   # WebView build is hooked into MSBuild
```

WebView (`src/Corsinvest.VisualStudio.Agents/Chat/WebViewSrc/`): `npm run build` / `dev` /
`typecheck` / `lint`.

**The startup project is `Corsinvest.VisualStudio.Agents`, and no file in git holds that.** VS keeps
it in the per-user `.vs/…/.suo`, and `.slnx` has no field for it — so adding a project can silently
leave F5 launching something else, and the fix is to set it back, not to look for a file to commit.

**Installs stack up.** VS keys extensions by `Identity Id`, so a build with a changed identity —
or a changed display name — installs *alongside* the old one: duplicate menu entries, two MCP
servers, and symptoms that look like bugs in the code. `tools\extension.ps1` is the test cycle
(remove every copy, then install into the Exp hives); `-Uninstall` clears them and refreshes the
hives, which deleting the folder alone does not — VS keeps serving the cached menu entries.

**Unit tests exist, and cover less than the extension does.** `tests/Corsinvest.VisualStudio.Agents.Tests`
holds ~195 xUnit tests over the pure logic — the JSONL readers, the content-block translator, meta
injection, stats, schema building. Run them:

```powershell
dotnet test tests\Corsinvest.VisualStudio.Agents.Tests\Corsinvest.VisualStudio.Agents.Tests.csproj
```

**Build first, then test — that order IS the gate.** The project takes a `<Reference>` on the built
`Corsinvest.VisualStudio.Agents.dll`, not a `ProjectReference` (a legacy VSIX resolves its
dependencies through packages.config, and the modern SDK rebuilding it buries the run in CS0246).
So `dotnet test` after editing a `.cs` and nothing else tests the PREVIOUS build — silently, and it
will look like your change is covered when nothing ran it.

Everything else — WPF, the WebView, the MCP surface, anything touching the VS shell — is verified by
hand in the Exp instance (F5 → `devenv /rootsuffix Exp`). A green build proves less than it looks:
XAML `x:Class`, `.vsct` ids and the manifest fail at *runtime*, not compile time — a mismatched
`.vsct` id gives a silent no-op menu entry. **CI does not run the tests**, so a red suite reaches
master unless someone ran it.

`tests/LangMatrix` is not a test project: five throwaway libraries the solution loads but never
builds, so the `nav_*` tools can be pointed at a real file in each language.

## Traps

Things that break in ways the compiler won't tell you about:

- **New `.cs` files must be added by hand to `<Compile>` in the `.csproj`** — explicit items, no
  glob. A file that compiles in VS can be silently missing from the MSBuild VSIX.
- **`CLAUDE_CODE_ENTRYPOINT=claude-vscode`** is mandatory when launching the CLI: without it
  `initialize` returns a reduced payload (no Fable / `unavailable_models`).
- **`Newtonsoft.Json` pinned to 13.0.1** — the version VS forces at runtime; a higher one throws
  `MissingMethodException`. Use `JsonExtensions.ToIndentedString`, not `JToken.ToString(Formatting)`.
- **Target framework v4.8**, not 4.7.2 — required by `Community.VisualStudio.Toolkit`.
- **`bridge-messages.ts` is generated** from `Chat/Host/BridgeMessages.cs` by
  `WebViewSrc/tools/gen-bridge.mjs` (part of `npm run build`). The C# file is the single source of
  truth — never edit the `.ts`.
- **`~/.claude/` paths and `claude.exe` names are the CLI's contract**, not ours. `ClaudePaths`,
  `ClaudeClient` and `ClaudeInstall` are named after what they drive: leave them alone.
- **`AppConstants.AppId`** names `%LOCALAPPDATA%\Corsinvest\<AppId>\` (profiles, WebView2 profile,
  caches). Changing it moves the user's data.
- **`.gitignore` excludes `[Dd]ebug/`** with an explicit exception for `Mcp/Tools/Debug/`. Renaming
  paths without updating it silently untracks those tools.

## Architecture notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Corsinvest/cv4vs-agents](https://github.com/Corsinvest/cv4vs-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
