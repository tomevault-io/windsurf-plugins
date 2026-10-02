---
trigger: always_on
description: Guidance for Claude Code (and future Claude sessions) when working on Stampeded!.
---

# CLAUDE.md

Guidance for Claude Code (and future Claude sessions) when working on Stampeded!.

## What this is

A keyboard-driven desktop code-review tool: a PR or local branch is read as a diff with real
semantic navigation (go to definition, find references, hover docs), blame, CI state, test
results and coverage, in one Avalonia window. See `README.md` for the pitch.

## Tech stack

- **Avalonia 12** with the **Simple** theme (not Fluent), **AvaloniaEdit** for the diff views,
  **Dock** for the pane layout, **Markdown.Avalonia** for rendered descriptions.
- **CommunityToolkit.Mvvm** (`[ObservableProperty]`) for view models; `Dock.Model.Mvvm` `Tool` /
  `Document` for panes and documents.
- **Roslyn** (`Microsoft.CodeAnalysis.*`) for source semantics: two workspaces per review, head
  and merge base, so removed code stays navigable.
- **TextMateSharp** for syntax colours: VS Code's grammars and themes, so a review colours a
  language nobody wrote an editor grammar for. Painted per side and transferred onto the
  document rows (`SyntaxPainter`, `DiffSyntaxColors`) - a grammar is a state machine over
  consecutive lines, and a unified diff is consecutive on neither side. The editor's own xshd
  definitions answer for what the bundle does not carry, which is ILAsm.
- The menu bar is a **NativeMenu**, not a `Menu`: macOS puts it in the system menu bar and
  `NativeMenuBar` draws it inside the window everywhere else, hiding itself where the platform
  took it. A menu item is a model object rather than a control, so it takes no `x:Name` (each
  one the code-behind reaches carries a key as its `CommandParameter` instead), has no
  `ItemsSource`, and shows a gesture only as text unless `Gesture` is set - which on macOS
  registers a real key equivalent that fires before the focused text box sees the key.
- **Light and dark are the reader's choice** (View > Theme, kept in `theme.txt`; the desktop's
  preference only until one is made), and the switch is live. XAML follows the theme variant
  through `DynamicResource`; whatever is painted from code asks `ThemeManager.IsDarkTheme` per
  paint and has to be repainted on `ThemeChanged` - and anything *built* with its colours in it
  (a TextMate painter, a comment box) has to be rebuilt there. `ThemeManager` stays close to
  ILSpy's, so its dark-mode fixes can be carried over.
- **CliWrap** for every external process.
- Target framework `net10.0`. Nullable enabled, implicit usings, `TreatWarningsAsErrors`,
  central package management (a new `PackageReference` needs a `PackageVersion` in
  `Directory.Packages.props`), and `AvaloniaUseCompiledBindingsByDefault` - so a typo in a
  binding path is a build error, not a silent blank.

## Project layout

- `src/Stampeded.Core/` - everything that does not need a UI: git and pull-request-host access,
  diff and fold building, Roslyn hosting, the LSP client, the review store. No Avalonia
  reference; keep it that way.
- `src/Stampeded/` - the Avalonia app: panes, documents, controls, view models.
- `src/Stampeded.RoslynLsp/` - Roslyn as a language server, for reading C# out of process.
- `tests/Stampeded.Core.Tests/` - NUnit, covering `Stampeded.Core` only. The UI layer has no
  automated tests.
- `Stampeded.slnx` builds all four.

## Everything external is a CLI

`git`, `gh`, `az`, `dotnet`, `code` and `xdg-open` are the only ways out of the process, all
through `ExternalTool.RunAsync` (which logs the command, and on failure the first line of its
output - an exit code alone never says what went wrong). There are no API tokens of the tool's
own: auth, SSO and token refresh ride on the user's `gh auth` and `az login`. Keep it that way;
do not add an HTTP client for any host.

A language server is the one exception, because it is not a command with an exit code: it
starts once and answers until the review closes, over JSON-RPC on its stdin and stdout
(`Stampeded.Core/Lsp/`). Everything it does still reaches the log - the command line, the
requests that take a noticeable while, every line of its stderr.

`CliLog.Write` is the log sink the Log pane shows. Anything a user might have to explain to
someone else belongs in it.

## A pull request comes from a host, not from GitHub

Everything a review asks about a pull request - the open list, its branches and description, its
checks, merge state, posted comments and thread resolution, the reviews, a verdict, a merge -
goes through `IPullRequestHost` (`Stampeded.Core/PullRequests/`). There are two:
`GitHubService` over `gh`, and `AzureDevOpsService` over `az` with its `azure-devops` extension
(`az repos pr ...`, `az repos policy ...`, and `az devops invoke` for the REST surface the
extension has no verb for - the analogue of `gh api`).

Which one answers is decided once per workspace, in `PullRequestHosts.ForAsync`, from the remote's
URL: what `AzureDevOpsUrl` parses is Azure DevOps, anything else is GitHub on purpose - `gh`
also serves GitHub Enterprise hosts, which nothing here can enumerate, and a clone with no
remote behaves as it always did. `STAMPEDED_PR_HOST=github|azdo` overrides it.

The remote is not assumed to be called origin: `GitService.GetRemoteAsync` takes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icsharpcode/Stampeded](https://github.com/icsharpcode/Stampeded) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
