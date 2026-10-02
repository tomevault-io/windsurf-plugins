---
trigger: always_on
description: > The full release/NuGet mechanism (diagrams, pitfalls, integration checklist) is in
---

# AgentBridge — notes for coding agents

> The full release/NuGet mechanism (diagrams, pitfalls, integration checklist) is in
> **[docs-dev/RELEASING.md](docs-dev/RELEASING.md)** — read it before touching the release pipeline.

## ⚠️ Before ANY push or release — developer checklist

**Read and satisfy [docs-dev/RELEASE-CHECKLIST.md](docs-dev/RELEASE-CHECKLIST.md) before every
push or release** (the pre-push hook prints the reminder). It is short on purpose: the layout
rules the code enforces — no config/state json next to the executable (only SDK-generated
`agent.*.json`), persistent files under `PersistentData\` or the OS app-data folder, the app
must run when launched from any directory — must never regress. A Debug build of agent refuses
to start when a stray json sits next to the exe (AppConfig); treat that refusal as a blocker.
Do not push while any box is unchecked.

## Documentation: two types (READ BEFORE WRITING A GUIDE)

AgentBridge has **two distinct documentation sets** — they are physically separated and
treated differently by the build:

| Folder | Audience | Shipped? |
|---|---|---|
| `docs/` | **End users** of the distributed app (manual, TUI/API references, telegram, sip, autoupdate, installers, `sip-entry/`) | **YES** — copied to the build/publish output and into every release archive (`docs/` next to the executable) |
| `docs-dev/` | **Developers** working on the repository (architecture, release pipeline, TUI internals) | **NO** — repository only, never shipped |

Rules:

- **A guide read by the end user of the distributed app goes in `docs/`.** It is copied to
  the destination automatically by `AgentBridge.csproj` (the `None` items with
  `CopyToOutputDirectory` at the bottom of the file) — no extra step needed. That is the
  whole point: **if a guide is not in `docs/`, it never reaches the people who install the
  app.**
- **A document for developers only goes in `docs-dev/`.** Architecture, release pipeline,
  TUI-internals, tooling. It stays in the repository and must NOT be referenced by shipped
  user guides as a required read (a shipped `docs/` guide may link to a `docs-dev/` guide
  only as an optional "(developers, not shipped)" note).
- **`media/` is README showcase only** (demo gifs/mp4, screenshots used by the repository
  README) — never shipped.
- Before writing any guide, ask: *who reads this — the person who installs the app, or the
  developer who maintains the code?* User → `docs/`. Developer → `docs-dev/`.

### The GitHub wiki mirrors `docs/` (keep it in sync)

The public wiki at **https://github.com/Graphene-Lab/AgentBridge/wiki** is generated from
`docs/` by `.github/workflows/sync-wiki.yml` (runs on every push that touches `docs/` or
`tools/wiki/`, and on manual dispatch). The workflow clones the wiki repo, regenerates the
flat pages with `tools/wiki/sync-wiki.js` (each `docs/**/*.md` becomes a wiki page named by
its file basename; `INDEX.md` → `Home`; relative `.md` links are rewritten to the flat page
name), and pushes.

**Rule: the wiki is never edited by hand — edit `docs/` and let the workflow sync it.** Any
change to a user guide in `docs/` must be pushed so the wiki regenerates; do not make
divergent edits directly in the wiki repo (they will be overwritten on the next sync). When
you add or rename a `docs/` page, the sync picks it up automatically; just make sure the
links between guides use the file paths so the rewriter can map them to the wiki page names.

## Release gate: IsPrerelease flag

`AgentBridge.csproj` carries `<IsPrerelease>` (default `true`). It decides whether a GitHub
release is produced:

- `false` → version is the date-based `1.yy.MM.dd` (e.g. `1.26.08.09`); the tag
  `v1.yy.MM.dd` triggers the full release build (release.yml).
- `true` → the version gets a `-prerelease` suffix and the release workflow skips
  (`check-version` job guards the build matrix).

Set the flag to `false` only when the test cycles proved the version works; keep it `true`
while iterating. Do not change the version scheme — it is date-based like UISupportBlazor.

**Release wait — no monitoring needed.** After tagging `v1.yy.MM.dd`, the workflow waits for
today's dependency packages on nuget.org within a **global 30-minute window** (nuget.org's
official propagation time), then builds anyway with a warning for any package not published
today (that repo simply had no push — only changed repos publish; for an unchanged repo the
latest available version is identical to today's). There is nothing to check during the wait:
either the build starts immediately or it starts after at most 30 minutes. See
docs-dev/RELEASING.md "How the release wait works".

## Release pipeline

**Automatic (recommended):** the release runs by itself in GitHub Actions on every master push
whose commit has `IsPrerelease=false` in `AgentBridge.csproj` (see `release.yml`): it waits for
the dependency packages on nuget.org, builds the 5 platform archives and creates the GitHub
release — the tag `v1.yy.MM.dd` is created on the fly. Nothing else to run; works from any git
client. With `IsPrerelease=true`, or when today's tag already exists, the run is skipped.
The "Push AgentBridge" status-bar button does exactly this with one click (no confirmation):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Graphene-Lab/AgentBridge](https://github.com/Graphene-Lab/AgentBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
