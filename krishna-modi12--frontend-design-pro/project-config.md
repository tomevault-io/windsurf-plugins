---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`frontend-design-pro` is a **skill pack for AI agents**, not an application. The deliverable is a `.skill` archive (a zip of markdown + TypeScript examples) that a host agent unzips and reads. Nothing here "runs" in the usual sense except `demo/showcase/`.

The product's entire claim is that it is **verified rather than asserted**: every number in the docs is derived from a green gate chain, and every example is machine-checked against 61 constraints. When a change and a gate disagree, the gate is right. If a change requires relaxing a gate to land, the change is wrong.

Because of that claim, the standing posture here is **judge before building**. Read [`docs/REVIEW_PROTOCOL.md`](docs/REVIEW_PROTOCOL.md) at the start of a session: it carries the five-minute spot check, the list of things no gate can see, and the severity ladder. Every release this project has had to correct was corrected for prose, not for code.

## Commands

```bash
npm run gates        # python scripts/build_release.py --dry-run  — all 11 gates, builds nothing. THE check.
npm run build        # full gated release: gates + archive + smoke test + release notes
npm run typecheck    # Gate 3 only — tsc --noEmit strict over every example
npm run constraints  # Gate 5 only — 44 regex constraints over catalog/
npm run figures      # Gate 11 only — every documented count/token figure vs the filesystem
npm run figures:test # proof that Gate 11's patterns read the prose forms people write
npm run hooks:test   # proof that the pre-commit guard sees every dirty porcelain state
npm run evals        # 22 eval cases, self-test
npm run regression   # 16 synthetic parser-vs-regex divergence cases
npm test             # Gate 7's runtime half — 45 files, 232 tests, ~35s
npm run banner:check # the README banner still draws the current figures
```

**`npm run gates` alone is not enough before you push.** Two artifacts are
*generated* from the figures and checked only in CI, by re-rendering and
byte-comparing: the README banner (`.github/assets/router.svg`) and `home/`'s
data file (`home/lib/data.generated.json`, which carries the band, the depth
and every per-skill budget — `.github/pages/data.js` before `home/` replaced
the static page it fed). Gate 11 cannot police either — its patterns match
inside markup and JSON, but every hit dies in the forbid look-back window,
which in markup is attribute soup rather than sentence.

So any change that moves a figure leaves a green 11/11 locally and a red `gates`
job on the PR. The frontmatter migration that added `metadata:` nesting did it
twice: once for the banner, then again for the Pages data file after a merge
brought a reference-depth change in from `origin/main`. That same shape
recurred verifying `home/`'s own PR — a merge landed after `home/`'s figures
were already written, moving reference depth by a full sweep's worth in one
step, and the fix was the same two commands below, not a hand-edit. After any
figure sweep run both, and commit what they write:

```bash
npm run banner && npm run pages:data
npm run banner:check && npm run pages:data:check   # what CI will assert
```

Renderer-level checks, for when you touch anything under `demo/` or `home/`.
These need a browser and real vendor libraries, so they live in
`tools/screenshots/` (its own package.json, absent from the archive manifest).
**Every rendered surface this repo publishes is now checked in CI, and both
jobs are blocking.** `home-renderer` runs `pages:verify`; `demo-renderer` runs
`demos:verify` and then `showcase:verify`, which are two commands because the
harness is split — `demos:verify`'s `ROUTES` mounts the three stub-typed demos
inside the `tools/screenshots` app and never touches `demo/showcase`, a
standalone app with its own install. Run only the first and you skip the demo
with the WebGL hero and the only real interactive flow in `demo/`.
`demos:typecheck` runs in `demo-renderer` too, ahead of the browser
download so a type error costs no Chromium install. **`screenshots` is the
one that stays manual** — it writes files rather than checking them, so CI
has nothing to assert about it that a reviewer's eye does not:

```bash
npm run demos:verify     # page errors · console errors · hydration · axe WCAG 2.1 AA
                         # · overflow at 390/768/1920 — dev AND production, both schemes
npm run demos:typecheck  # the demos against REAL vendor typings, not demo/_stubs.d.ts
npm run showcase:verify  # demo/showcase: the same, plus the pricing-to-contact flow
npm run pages:verify     # home/: the same renderer checks, dev AND production, plus
                         # whether the router and the checker do what the copy says
npm run screenshots      # regenerate every image README.md links
```

`pages:verify` starts `home/`'s own dev and production servers — it needs the
app built (or `npm install`ed for dev), unlike the static page it replaced. It
is worth knowing what the equivalent check found the first time it ran against
`home/`, because none of it was visible in source: `text-accent` on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Krishna-Modi12/frontend-design-pro](https://github.com/Krishna-Modi12/frontend-design-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
