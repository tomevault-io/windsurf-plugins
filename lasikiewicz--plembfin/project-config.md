---
trigger: always_on
description: Agent instructions for working with this codebase.
---

# CLAUDE.md

Agent instructions for working with this codebase.

> **Before changing anything, read [`docs/architecture.md`](docs/architecture.md).**
> It is the master guide: the big picture, a complete map of every file in the repo,
> request flow, the data layer, auth, and environment variables. The table at the end
> of this file routes you to the doc that owns the area you are touching.

> **For website update requests, also follow [`docs/websiteupdate.md`](docs/websiteupdate.md).**
> Start by discovering the verification baseline recorded by the website, then review
> and document all changes after that baseline and visually check affected pages locally.

> **For website publishing setup, follow [`docs/website-deployment.md`](docs/website-deployment.md).**
> The website is a static Astro site in `website/`; do not push or deploy it without an
> explicit user request.

> **IMPORTANT — website/Traks work is isolated and this rule is mandatory.** Read the
> website deployment docs and skill before acting; use only the local `website/` tree and
> direct Wrangler deployment to `plembfin-website`. Never use GitHub, modify `main`, run
> application CI, run the root Plembfin build, or touch the `plembfin` application project
> unless the user explicitly requests **"Force to main"**.

> **For the explicit phrase "Push website live", follow
> [`.claude/skills/push-website-live/SKILL.md`](.claude/skills/push-website-live/SKILL.md).**
> This is a website-only publish from the current local `website/` tree to the separate
> `plembfin-website` Cloudflare Pages project; it must not push GitHub or rebuild the
> Plembfin application.
>
> **For the explicit phrase "Start the website", follow
> [`.claude/skills/start-website/SKILL.md`](.claude/skills/start-website/SKILL.md).**
> This starts the local Astro preview at `http://localhost:4321/` for editing and
> testing only; it must not publish or deploy anything.
>
> **For the explicit phrase "Check requests", follow
> [`.claude/check-requests.md`](.claude/check-requests.md).** This is a local-only,
> read-only monitor for the Plembfin feature requests on Plex, Emby, and Jellyfin;
> it reports status, votes, comments, and any reply that needs a draft.

## Local testing context

When local-server or connected-browser testing is explicitly requested, check whether
`.claude/local-environment.md` exists and read it before testing. It contains
machine-specific local URLs and signed-in browser-session context, and is intentionally
gitignored. Treat it as optional because it is absent from fresh clones and other
machines.

## Agent Guidelines

- **No Git Pushes** - Never execute `git push` or push commits to any remote repository unless the user explicitly instructs you to push in their request.
- **No Deployments** - Never deploy the application or run deployment commands unless explicitly instructed by the user.
- **No Unsolicited Actions** - Do only exactly what the user asks. Do not perform unsolicited refactorings, add extra features, or modify files outside the direct scope of the request.
- **No Browser Actions Unless Asked** - Never open web browsers/browser tools unless the user has explicitly requested it. Test commands are part of the normal project checks: run `npm test` or `npm run build` when a change touches code covered by those checks or when the user asks for verification.
- **Website-only safety** - Website/Traks work must use the local `website/` checks/build and direct deployment to `plembfin-website`; it must never modify GitHub `main`, trigger application CI, run the root build, or touch the `plembfin` application project. The only exception is the explicit release workflow **"Force to main"**.
- **Act immediately on simple requests** - If the user describes a clear, specific change, make it directly without preamble, planning steps, or explanation. Save analysis for genuinely complex or ambiguous tasks.

## Branching model: `develop` → `alpha` → `main`

Day-to-day work lands on the `develop` branch, never directly on `alpha` or `main`.
`alpha` only moves when the separate "Force to alpha" command explicitly promotes
`develop` onto it; `main` only moves when "Force to main" explicitly promotes `alpha`
onto it, and each promotion to `main` becomes exactly one release (one changelog entry,
one version bump, one `:latest` + versioned Docker image publish).
After a main promotion nothing is pushed back: the release is not merged into `develop`
and `origin/develop` is not touched. Each promotion writes the new version numbers into the
local manifests and the next ordinary "Push to git" publishes them. This is what keeps a
promotion to one channel from also publishing a second, meaningless develop image. See
`docs/decisions.md` entry 18, which supersedes entries 7, 9, and 16.

### Changelog content is computed locally, before every push - never by CI

All three of `changelog.develop.json`, `changelog.alpha.json`, and `changelog.json` are
written locally, by the push command that produces them, from real local git history.
No CI job ever writes changelog content or a version back to a branch; the publish
workflows only build, verify, and publish using values already committed.

The reason: `alpha` and `main` are always reached by force-push, and a force-push's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lasikiewicz/plembfin](https://github.com/Lasikiewicz/plembfin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
