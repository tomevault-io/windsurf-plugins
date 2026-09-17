---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## What this is

`manygit` — a lazygit-style terminal UI for a whole **tree** of git repos. Point it
at a folder; it walks three levels down (skipping `node_modules`, `vendor`, `dist`,
…), groups every repo it finds by parent, and lets you fetch / pull / push /
checkout the one under the cursor. Go 1.24, Bubble Tea + Lip Gloss, single binary.

Published to GitHub Pages from `docs/` → <https://rabeeh-ta.github.io/manygit/>.

## The rule: the landing-page demo mirrors the TUI

**`docs/` contains a working browser port of the TUI. When you change a feature in
the Go CLI, update the demo in the same change.** It is not a screenshot or a
recording — it is a real reimplementation of the interaction model, and the page
tells visitors "the keys are the real keys". A demo that drifts from the binary
makes the page lie.

This applies to: a new/changed keybinding, a new pane or tab, a renamed pane, a
changed status glyph, a new theme, a new settings row, a changed empty/error
state, or new copy in the footer/help.

### Where things map

| Go (source of truth) | Browser port |
|---|---|
| `internal/tui/update.go` → `handleKey` | `docs/assets/demo.js` → `handleKey` |
| `internal/tui/view.go` → `syncGlyph`, `renderRow`, `tabBar`, `overlayTabs`, `overlayHead`, `window`, `centerBlock` | `demo.js` → same names, ported deliberately |
| `internal/tui/theme.go` → `themeList` | `docs/assets/site.css` → `:root[data-theme=…]` blocks |
| `internal/tui/settings.go` → `settingRows` | `demo.js` → `settingRows` |
| `internal/discover` → repo/script discovery | `demo.js` → the `REPOS` / `SCRIPTS` fixtures |
| `README.md` key table | `docs/index.html` → the Keys section |

Ported functions keep their Go names on purpose — grep the name in both files.

**There are FOUR mirrors of the key table, not two.** A key change has to land in
all of them or the product and the site disagree:
`internal/tui/view.go` → `keysBody` (the in-app reference), `README.md`,
`docs/index.html` → the Keys section, and **`docs/llms.txt`** — which is
git-tracked and published from `docs/` alongside the page, and is the one people
forget.

Two more move with a key change and are just as easy to miss: `view.go` →
`footer()` (the always-visible hint strip, mirrored in `demo.js`'s
`statusOrFilter`), and the **Safety** sections in `README.md`, `docs/index.html`
and `docs/llms.txt` — those make claims about what manygit can do to your repos,
so a key that widens what it can do has to be reflected there or the page is
lying. Adding `!` is what surfaced this.

### Rules the port must keep

- **Copy strings verbatim** from the Go where the demo shows one (empty states,
  status messages, hints). If the Go says `You're all caught up`, so does the demo.
  Don't "improve" it in the port — change the Go, then re-port.
- **Themes are chrome only.** `theme.go` themes `accent/group/dim/error` and
  nothing else; the page does the same. Status colours (`ok`, `↑N`, `*N`, …) stay
  fixed across themes, per theme.go's own comment.
- **`--dim` is not a prose colour.** It's the terminal's chrome colour and fails
  WCAG AA on this background in several themes. Page text uses `--muted`, which is
  each theme's `--dim` lifted to ≥4.5:1. Terminal-internal text keeps `--dim`.
- **Two independent axes: `data-mode` × `data-theme`.** `data-mode` (light/dark)
  is the *ground* and belongs to the site, because manygit has no background
  setting — it inherits your terminal's. `data-theme` is the *chrome* and is
  manygit's. They compose: six themes × two grounds = twelve palettes, and every
  one must clear AA. A new theme means adding **both** a
  `:root[data-theme=…]` block and a `:root[data-mode="light"][data-theme=…]` one
  — theme.go's accents are tuned for a dark terminal and none of them pass on
  paper unmodified. Darken hue-preserving and **measure**; don't eyeball.
- **The demo intentionally diverges in exactly five places**, all because it runs
  in a browser:
  1. `q` explains itself instead of quitting.
  2. `o` explains itself instead of spawning an editor.
  3. **`esc` releases the keyboard when it has nothing else to do.** The widget
     swallows `tab` *and* `shift+tab` (both cycle panes), so without an exit a
     keyboard user is trapped — WCAG 2.1.2. `esc` keeps its real meaning inside
     the Changes pane and any overlay/filter/confirm; only the otherwise-inert
     case is spent on blurring. If you rebind `esc` in the Go, find the demo a new
     exit **and say so in the `aria-label` and the visible cue** — an undisclosed
     escape hatch is the same as none.
  4. **`:` replies from a canned script instead of an AI harness.** A browser has
     neither `claude` nor git, so `cannedPlan()` recognises a few request shapes
     and returns a plan for them. Everything *around* the fake is real and ported:
     the input, the ghost-text completion, the validator refusing a force-push,
     the confirm, and stop-at-first-failure. The page says the reply is scripted,
     the same way it says the git is fake — if you extend `cannedPlan`, keep that
     label true and never let it imply the model is actually running.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rabeeh-ta/manygit](https://github.com/rabeeh-ta/manygit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
