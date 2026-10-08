---
trigger: always_on
description: > C# / .NET 10 / Avalonia 12 BBS terminal client. Linux primary, cross-platform via Avalonia.
---

# MudPlay — Claude Code Instructions

> C# / .NET 10 / Avalonia 12 BBS terminal client. Linux primary, cross-platform via Avalonia.
> This file is **workflow guidance**, not architecture documentation. Subsystem invariants live in code comments next to the code they constrain. If a rule here conflicts with what the code actually does, the code wins and this file gets updated.

## Project at a glance

MudPlay is a Telnet-based BBS terminal client. It speaks Telnet (RFC 854/855 with NAWS and TERM-TYPE), parses VT100/ANSI escape sequences in `Terminal/TerminalEmulator.cs`, renders a CP437 cell grid with a custom Avalonia control in `Controls/TerminalControl.cs`, and uses MVVM (CommunityToolkit.Mvvm source generators — `[ObservableProperty]`, `[RelayCommand]`).

What it is **not**: not a PTY wrapper, not a generic SSH client, not a curses-style application terminal. All ANSI parsing is explicit; we don't rely on a host TTY.

## Workflow

The phased implementation plan is **complete** — the app is at **1.0.0** and in a bugfix / maintenance cycle. There is no more `docs/` build plan; work now flows as focused fixes and small enhancements, each on its own branch + PR.

- **PR cadence**: one open PR at a time; the next doesn't begin until the current merges. **Bug reports dropped together all land on the same open PR** — a batch shares one branch + PR (consolidated changelog entry, PATCH counting the reports per the versioning rule below). The exception is a report that needs **more than a fix to resolve** — a significant feature-scale design change or new implementation: **suggest breaking that onto its own PR and let the user decide** (they'll agree, or tell you to fold it into the current open PR). Keep each PR focused (see PR-size discipline under Scope discipline). The user merges the PR when it looks good to them; after the merge, wait for whatever comes next. **You never merge or close a PR yourself, and you never auto-open a *follow-up / split* PR — those are the user's calls (see the auto-commit rule below).**
- **Auto-commit + push when a batch is done — don't wait to be asked.** The default "never commit unless explicitly asked" holds *mid-work*, but the moment a batch of fixes/feature is genuinely finished — `dotnet build` clean, full `dotnet test` green, and the version-history trio (csproj `<Version>`, CHANGELOG top entry, README current-version block) updated — commit it (with the `Co-Authored-By` trailer) and push on your own. **Opening a PR autonomously is OK only when the user handed you a discrete new work item** — a bug report (or batch of them), or a feature / fix / update to the client — that fresh item gets its own PR. **When the work is a follow-up to, or a split of, the PR you're already in, do NOT open a new PR on your own: the user decides whether it folds into the current PR or gets a new one — wait for that call.** If a PR already exists for the branch, confirm it's still open (`gh pr view <n> --json state`) and push to it. **Never close or merge a PR unless the user explicitly said to.** Never auto-commit a red build / failing tests / half-finished work — this is the *closing* move on a completed batch, not a running habit. **After each commit, rebuild the user's debug local so what they run reflects the latest** — which is why the working branch lives on their main checkout, not an isolated worktree.
- **Versioning (semver, post-1.0)**: the version lives once in `MudPlay.csproj` `<Version>` (`AppInfo.Version` reads it back for the `@version` reply). Bump it by **change type, not PR size**:
  - **MAJOR** — a whole-program refactor or other sweeping/breaking overhaul. Rare.
  - **MINOR** — a **brand-new feature** (a capability that didn't exist before), **or a large rewrite / expansion of an existing feature** (roughly **1000+ lines net**). One bump per such change, and it resets PATCH to 0 (e.g. 1.5.11 + a new feature → 1.6.0).
  - **PATCH** — bug fixes **and ordinary enhancements to existing features** (a changed default, a tweak, a smaller improvement — anything under the ~1000-line MINOR bar). **Increments by the number of bug reports handled plus the number of enhancements** — one report or one enhancement on 1.5.11 lands 1.5.12; a batch of five handed over together advances 1.5.11 → 1.5.16. An ordinary enhancement counts exactly like a bug report: it moves the third number, never the minor.

  A PR that mixes types takes the highest (a new feature or a 1000+-line expansion plus fixes/enhancements is a MINOR; fixes plus ordinary enhancements stays a PATCH). The version counts the reports + enhancements, but the changelog does **not** need one entry per item: a batch shares a single consolidated `## <final-version>` entry whose bullets cover all the changes.
- **Every PR updates the version history.** `CHANGELOG.md` is the running record, newest entry first. On each PR, in the same branch:
  1. Bump `<Version>` in `MudPlay.csproj` per the semver policy above.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tehshortbus/MudPlay](https://github.com/Tehshortbus/MudPlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
