---
trigger: always_on
description: Guidance for working in this repo. Hard-won - read before changing CSS, raising PRs, or touching Supabase.
---

# CLAUDE.md

Guidance for working in this repo. Hard-won - read before changing CSS, raising PRs, or touching Supabase.

## UI conventions

- Icons come from `lucide-vue-next` (already a dependency) - never emoji glyphs in UI chrome. Country flag emoji on leaderboards/profiles are the one exception.
- **Never hand-roll an icon as inline `<svg>`, and never stand one in as a text glyph or HTML entity** (`&rarr;`, `&times;`, `✕`, `✓`). Import the Lucide component. Hand-drawn duplicates of Lucide shapes and entity arrows were the whole of the 2026-08 icon cleanup.
- **Icon scale is fixed: `:size="14"` inline-with-text, `16` buttons and chrome, `20` icon-only buttons and section headers, and `:stroke-width="2"` always.** Only large display/empty-state icons may sit outside it (currently two at 40). Deviating is what made the UI read as sloppy - there were six sizes and five stroke widths.
- Legitimate inline `<svg>` is limited to: brand marks Lucide does not carry (Google, GitHub, X, WhatsApp - prefer `simple-icons` paths), custom artwork (`CardBack`, `AutoStartRing`, the landing strike line), and genuinely two-tone icons Lucide cannot express (the `VoiceMicCluster` mic, whose slash is styled `--color-alert` separately).
- Concerns are color-zoned with the deck palette: hazard yellow = daily/streak loop, alert red = primary create action, neon cyan = multiplayer, neutral = practice/meta.

## CSS tokens

- The spacing scale in `frontend/src/style.css` is `--spacing-0..4`, then jumps to `6, 8, 12, 16, 24`. **There is no `--spacing-5`** (or 7, 9-11, etc.).
- An undefined CSS var is silently invalid: `gap: var(--spacing-5)` collapses to 0, and in a shorthand (`padding: var(--spacing-4) var(--spacing-5)`) the whole declaration dies - zero padding.
- The source looking right proves nothing. After using any token you haven't confirmed exists, verify the computed style in a browser (`getComputedStyle` or devtools), not the stylesheet.

## PR workflow

- Feature work goes on a branch with a PR into `main`. Cloudflare Pages auto-deploys `main`.
- **Every PR gets reviewed with the repo skill before merge**: run `.claude/skills/github-pr-review` (trigger: "review this PR"). It posts findings as inline comments on specific lines and reads `.claude/review-patterns.md` as a required repo overlay - the incident-earned checklist lives there. Then close the loop per that overlay: fix every Critical and Major finding, push the fixes to the same branch, and resolve each addressed comment thread (reply with what changed, then resolve). A PR with unresolved review threads is not merge-ready.
- **Stacked-PR merge trap**: GitHub only retargets a stacked PR to `main` when its base branch is deleted after the base PR merges. Merging a stack quickly without deleting branches makes each PR merge into its original base branch - `main` gets only the bottom of the stack and the rest strands on feature branches, silently.
  - Prefer PRs based directly on `main`.
  - If you must stack: delete each branch as its PR merges, and verify `git log origin/main` actually contains the work afterward.
- Run `npm run build` (from `frontend/`) and `npx vitest run` before pushing. Build = `vue-tsc -b && vite build`, not just typecheck.

## Shipping updates to players

- Every user-facing change ships its changelog entry **in the same PR** as the change. `frontend/src/data/changelog.ts` is the single source; the panel, the release card, and `/changelog` all read it. An entry that lands in a later PR announces a feature that is already old.
- **Ask the human which volume before you open the PR. Never pick it alone.** Two options, and the answer goes in the entry's `level`:
  - `quiet` - the entry appears in the What's New panel and puts a dot on the top-bar link. This is the default. Use it for anything a player does not need to be told about today.
  - `loud` - the entry also fires a one-time release card over the lobby. Reserve it for a change that alters what a player does. At most one per quarter. A card on every release trains people to close it unread, which kills the channel for the release that needs it.
- Entry shape: `{ id, level, tag, title, body, cta? }`. `id` is the ISO date (`2026-08-26`) and must sort. `tag` is `NEW`, `IMPROVED`, or `FIXED`. `title` is verb-first and under 50 characters. `body` is one or two sentences in plain words. `cta` is optional and must route somewhere in `utils/routes.ts` - no link is better than a link to the home page.
- The dot is driven by the newest `id` against a last-seen id in `localStorage`. Adding an entry is what makes the dot appear, so never add one for a change that has not deployed.
- **Both surfaces are for players, not visitors.** The panel lives in the lobby top bar and the card renders only when signed in, so someone who has never played is not told what changed in a product they have not used. A guest who plays is signed in, so "signed in" means "has played". `/changelog` stays public: a page someone navigates to is not a nudge.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alii13/open-mercy](https://github.com/alii13/open-mercy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
