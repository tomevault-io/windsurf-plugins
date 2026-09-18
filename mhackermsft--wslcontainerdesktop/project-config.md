---
trigger: always_on
description: A native **WinUI 3 / .NET 10** desktop app (Docker-Desktop-like) that manages **WSL containers**
---

# WSL Container Desktop — Copilot instructions

A native **WinUI 3 / .NET 10** desktop app (Docker-Desktop-like) that manages **WSL containers**
via the `wslc.exe` preview CLI, a single-node **k3s** cluster inside WSL, and container registries.
It is a packaged (MSIX-identity) app that minimizes to the system tray.

For deep design detail read `docs/ARCHITECTURE.md`; user-facing features live in `README.md`.

## Dependency policy (supply-chain security)

**Never add or upgrade to a dependency version published less than 7 days ago.** A just-released
version is exactly when a compromised, hijacked, or typosquatted package is most likely to be live
and least likely to have been reported and pulled. Age is therefore treated as a security control
in its own right, independent of how reputable the publisher is or how badly a fix is wanted.

- **Scope is every dependency**, not just NuGet: direct, transitive, and build/test-only packages,
  plus GitHub Actions, CLI tools, container images, and scripts the build or tests fetch.
- **Verify the publication date from an authoritative source** — the nuget.org registration index
  or the upstream release feed — *before* restoring it. "Looks established" is not evidence, and a
  package's own build timestamp is not its publication date.
- **If the date cannot be verified, stop and say so.** Never assume compliance. Waiting a few days
  is always cheaper than tracing a compromised dependency through a shipped build.
- **Prefer what is already referenced.** Adding a package needs the same justification as any other
  change; it is not a free shortcut. Most feature work here needs no new dependency at all.
- A version that is otherwise required but too new is a **blocker to raise**, not a judgment call to
  make silently. (This mirrors the MSIT dependency-age requirement.)

## Environment & build

- **Requires Windows 11** with the WSL container preview (`wslc.exe`, default
  `C:\Program Files\WSL\wslc.exe`) and the **.NET 10 SDK**. The app cannot fully build on Linux —
  the WindowsAppSDK XAML compiler step requires Windows.
- Work from `src\WslContainerDesktop`. Always target the **x64** platform.
- **Build:** `dotnet build -c Debug -p:Platform=x64`
- **Run (dev):** `dotnet run -c Debug -p:Platform=x64`. Use `dotnet run`, **not** the bare
  `bin\...\WslContainerDesktop.exe` — this is a packaged app; running the raw exe crashes at
  startup with `REGDB_E_CLASSNOTREG`. `dotnet run` registers the debug MSIX identity, refreshes the
  loose layout, and launches with package identity. A plain `dotnet build` leaves the *registered*
  app pointing at a stale layout.
- **Fast dev loop:** `tools\launcher\Build-And-Run.ps1` rebuilds, redeploys, and launches in one step.
- **Release:** `.github/workflows/release.yml` (manual `workflow_dispatch`) builds a signed MSIX;
  publish profiles live in `Properties/PublishProfiles/`. **Update `CHANGELOG.md` first** — see
  [Releasing](#releasing) below.
- **Tests:** `dotnet test tests\WslContainerDesktop.Tests\WslContainerDesktop.Tests.csproj -c Debug -p:Platform=x64`
  from the repository root; use focused filters for changed behavior. Build to **0 warnings**.
  Coordinate packaged smoke runs: deployment affects the registered app, even from another worktree.

## Releasing

`CHANGELOG.md` is the user-facing record of each release, and the generated release notes link to
it at the release's own tag. **Update and commit it before starting the release workflow**: the tag
is created from the repository as it stands, so a changelog written afterwards leaves that release's
link pointing at an entry that does not mention it.

Between releases, accumulate entries under a `## [Unreleased]` heading at the top. Add to it as
user-facing work lands rather than reconstructing the whole release at the end — that is when the
detail is still known, and it keeps an unreleased entry from claiming a version and date that do
not exist yet.

Steps, in order:

1. **Promote `[Unreleased]` to the new version** — `## [X.Y.Z] — YYYY-MM-DD`, newest at the top —
   and update the link list at the bottom of the file: point `[X.Y.Z]` at
   `…/releases/tag/vX.Y.Z`, and re-point `[Unreleased]` at `…/compare/vX.Y.Z...main`. Keep the list
   in the same newest-first order.
2. **Group entries under the headings that apply** — `Added`, `Changed`, `Fixed`, `Removed`,
   `Deprecated`, `Security` — and omit headings with nothing under them.
3. **Write for someone using the app**, not someone reading the diff. State what they can now do,
   what behaves differently, or what was broken and now isn't; when a fix is non-obvious, say what
   the user would have seen. Leave out pure refactoring, test-only work, and internal renames unless
   they change observable behavior or a default.
4. **Call out anything that changes an existing default** (a template's published port, a provider
   default, a settings value), because that is what silently breaks a working setup.
5. **Derive entries from the actual commit range** (`git log --no-merges vPREV..HEAD`), not from
   memory. Verify a claim before writing it; an inaccurate changelog is worse than a terse one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhackermsft/wslcontainerdesktop](https://github.com/mhackermsft/wslcontainerdesktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
