---
trigger: always_on
description: This repo packages a Maktabah al-Shamela 4 search server as an `.mcpb` (MCP Bundle) for install into Claude Desktop. Architecture, IPC contract, citation format, and roadmap live in [docs/](docs/).
---

# shamela-mcpb — Claude Code Context

This repo packages a Maktabah al-Shamela 4 search server as an `.mcpb` (MCP Bundle) for install into Claude Desktop. Architecture, IPC contract, citation format, and roadmap live in [docs/](docs/).

## Build commands

All scripts are Node-based (no PowerShell) so they run on Windows + macOS + Linux:

```bash
npm install                 # one time per checkout
npm run build               # esbuild Node + javac Java helper
npm run test                # unit + integration suite (vitest)
npm run smoke               # exercise every tool against the local Shamela install
npm run benchmark           # Mode 1 + Mode 2 workflow simulations
npm run pack                # full chain: build:server + build:java + mcpb pack
npm run release             # cut a release: pre-flight + pack + tag + GitHub Release
npm run release:dry         # run all pre-flight checks, skip pack/tag/publish
```

`build:java` requires JDK 21+ (`javac` + `jar`). It searches PATH, then `JAVA_HOME`,
then platform defaults: Eclipse Adoptium / Microsoft / Oracle / Corretto on Windows,
`/usr/libexec/java_home -v 21` on macOS, `/usr/lib/jvm/*` on Linux. Set `JAVA_HOME`
explicitly if your JDK is in a non-standard location.

## Release workflow

Releases publish a `.mcpb` to GitHub Releases on this repo. The flow is
**manual on the developer machine** — CI can't build the helper jar
without the user's Shamela install (which we can't ship for clean-room
reasons).

### When the user says "ship a release" / "cut a release" / "publish"

**Claude decides the semver bump using the rules below.** Don't ask the user
which bump to use unless the change is genuinely ambiguous. State the chosen
bump and rationale in one sentence before proceeding, so the user can override
if they disagree.

The release is one command: `npm run release`. Before running it, Claude
must do the version bump:

1. Read the current version from `manifest.json` (single source of truth).
2. Decide the bump per the rules below; compute the new version `X.Y.Z`.
3. Update **both** `manifest.json` and `package.json` to `X.Y.Z`.
4. `git commit -am "release: vX.Y.Z — <one-line summary>"` and push.
5. `npm run release`.

`npm run release` runs eight pre-flight checks (clean tree, on main, in sync
with origin, version consistency, tag unused, commits since last tag, vitest
green, gh authenticated) and refuses on any failure. After pre-flight: packs
the `.mcpb`, creates an annotated tag, pushes it, and publishes a GitHub
Release with the `.mcpb` attached and auto-generated notes from commits since
the last release.

### Semver decision rules (Claude follows these autonomously)

Inspect every commit since the most recent `v*` tag
(`git log <last-tag>..HEAD --oneline`) and pick the **highest** category
that any commit falls into:

**MAJOR (X.0.0)** — breaks existing callers. Every user must update.
- Removed a tool from the registered set
- Renamed a tool
- Removed an input parameter, OR added a new REQUIRED input parameter (no default)
- Removed a field from `structuredContent`, OR changed an existing field's type
- Changed tool semantics so the same input now returns materially different results
  (e.g. a search returns different hits, a citation formats differently)

**MINOR (x.Y.0)** — adds capability, backward compatible.
- Added a new tool
- Added a new optional input parameter (with a sensible default)
- Added a new field to `structuredContent` alongside existing fields
- New search options, new scope filter dimensions, new citation styles
- Performance improvement that meaningfully changes latency tier (e.g. 10s → 1s)

**PATCH (x.y.Z)** — fix or invisible-to-user change.
- Bug fix that makes the tool behave the way the docstring already promised
- Documentation / docstring text update (LLM-facing prompts count — they
  don't change the JSON API)
- Internal refactor, test additions, build/CI/script changes
- Performance improvement with no observable difference to the caller
- Dependency bumps (without behavior change)

**No bump needed at all** — don't release.
- Only docs/test/build commits, AND no fixes to code that ships in `dist/`
  (e.g. README typo, CLAUDE.md edit, GitHub Actions tweak)
- Note: even one bug-fix commit triggers at least a patch release.

**Edge cases — ask the user instead of guessing:**
- A "fix" that changes behavior in a way some users could reasonably have
  depended on (e.g. today's strict `downloaded` flag — books that used to
  report `true` now report `false`). Argue patch (it was a bug), but flag it.
- A new tool that supersedes an old one but the old one isn't removed yet —
  could be minor or could be major-with-deprecation.
- Anything where the commit messages are unclear about user-visible impact.

### Worked example — today's situation (post-1.0.0)

Commits since `v1.0.0` (none yet, so since the latest `main`):
- `test:` install vitest pyramid → patch (test infra, no shipped code)
- `fix:` search_books scope → patch
- `fix:` get_book.downloaded strict → patch (was a bug — flag to user since
  it's the borderline case)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alhoqbani/shamela-mcp](https://github.com/alhoqbani/shamela-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
