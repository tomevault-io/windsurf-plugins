---
trigger: always_on
description: An AI-native, keyboard-first Git desktop client (Tauri 2 + React 19), with an Astro
---

# GitDesktop — agent guide

An AI-native, keyboard-first Git desktop client (Tauri 2 + React 19), with an Astro
marketing site in `site/`. This file is the standing brief for Claude/agents; the
full human conventions live in [CONTRIBUTING.md](CONTRIBUTING.md), product intent in
[PRODUCT.md](PRODUCT.md), and ongoing project state in `memory/` (auto-loaded).

## Keep docs in sync with features — every time, unprompted

**When you add or ship a user-facing feature, update its docs in the SAME change —
don't wait to be asked:**

1. **`README.md`** — add/extend the relevant bullet under *Highlights* and/or *Features*.
2. **Marketing site** — add the capability to `site/src/data/capabilities.ts`, the
   single source of truth for the catalog (set `ai: true` only for AI features;
   `highlight: true` to surface it on the home page). Then add or extend a
   `FeatureRow` in `site/src/pages/index.astro` when it warrants a section. That
   page has two synced views, **AI-native** and **Just Git** — put non-AI features
   in both, AI features in the AI view only. Then `cd site && pnpm build` to verify.
3. **In-app user guide** (`src/features/help/content.ts`) — when a change adds or
   meaningfully alters a user-facing surface, update the matching guide section (or add a
   new one for a whole new surface), and keep it accurate (verify claims against the
   code, not memory). Shortcuts are `{{kbd:action-id}}` / `{{key:…}}` tokens, **never
   literal keys** (they resolve per-platform and reflect rebindings); gate AI content with
   the `ai: true` section flag + `{{ai}}…{{/ai}}` inline markers so *Hide AI* hides it.
   (Conventions + gotchas: `memory/help-guide-content-conventions.md`.)
4. **Changelog fragment** — for any user-facing **app** change, add a
   `changelog.d/<added|changed|fixed>-<slug>.md` file whose body is the finished
   Keep a Changelog bullet (written for humans). The changelog ships in-app (release
   body + updater notes), so **site-only work gets no fragment** — steps 1–2 are its
   record. **Never edit `## [Unreleased]` in `CHANGELOG.md` directly** — fragments
   are assembled there at release time, and one file per change keeps parallel
   branches conflict-free. Preview with `pnpm changelog:preview`; conventions live
   in `changelog.d/README.md`. CI keys on paths: the required `fragment` check
   fires on any `src/` or `src-tauri/` change; one that needs no fragment takes
   the `no-changelog` label or `skip-changelog` in the PR title.

When a change alters **existing** behavior, grep README / site / help for the old
wording (e.g. the feature's old phrase) rather than updating spots from memory — stale
copies of the same claim hide across all three surfaces.

If a feature is too minor for the README / site / guide, it's fine to add only the
capability line + changelog fragment — but make the call deliberately, don't skip silently.

**Screenshots:** marketing-site screenshots for the **Just Git** view must be
captured with the app's *Settings → General → Hide AI features* ON, so they match
the AI-hidden experience. (See `memory/site-just-git-screenshots.md`.)

## Blog-post copy — anti-AI-tell rules (`site/src/content/blog/`)

Article prose ships under the owner's byline; readers (and the owner) flag AI-sounding
copy on sight. Hard rules for post text, learned the expensive way on PR #132 — they
govern phrasing only; technical claims stay empirically probe-verified as ever. They
apply to **every** post, shipped ones included: when a rule tightens, sweep the
back-catalogue in the same change rather than grandfathering it.

- **No earnestness performance.** Never "honest", "genuinely", "truly" as sincerity
  markers ("Two honest caveats"), and no valuation tics ("worth knowing", "worth the
  whole article", "it's worth").
- **No takeaway scaffolds.** "The thing to take away is…", "here's the part that…",
  "the key insight…" — state the point, don't announce it.
- **Antithesis budget.** "X, not Y" / "isn't X — it's Y" is a density guide, not a
  ban: keep every instance where the contrast IS the technical point being made, and
  cut the rest — reflex-reaching for the scaffold is the tell, especially twice in
  adjacent paragraphs.
- **Em-dash budget.** ≈7 per 1k words of running prose — list-item glosses, tables,
  and code don't count toward the rate. No paired "— aside —" interrupters; recast
  with commas or parentheses.
- **Closers written fresh every post.** The "Or don't do any of this" heading is a
  series motif; the prose beats under it are not. Never reuse a prior post's scaffold
  ("I write a Git client, so…", "Same X, same Y, minus Z", "Either way, the thing to
  take away…").
- **Measured numbers trace to transcripts.** Any figure you *measured* must be
  visible in a code block the reader has already seen — drop or demonstrate, never
  assert. Documented facts (version numbers, release dates, config defaults) may be
  cited directly.
- **Wrap band.** Prose lines ~76 chars, 81 max; no orphaned short lines
  mid-paragraph, no one-word paragraph trailers. Bare URL / link-target lines are
  exempt — never break a URL to satisfy the band.
- **Voice constants** (house style for the series): cold second-person open and a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theBGuy/GitDesktop](https://github.com/theBGuy/GitDesktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
