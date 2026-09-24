---
trigger: always_on
description: A single-file, offline, vanilla-JS outliner. Every point can generate (`{2d6}`) or compute (`{= sum(cost)}`). Everything lives in `index.html`. No build step, no network, no accounts.
---

# Pointliner

A single-file, offline, vanilla-JS outliner. Every point can generate (`{2d6}`) or compute (`{= sum(cost)}`). Everything lives in `index.html`. No build step, no network, no accounts.

## Commands

```bash
node --test tests/test.mjs      # the gate: pure cores + source pins + drift guards, offline
node --test tests/browser.mjs   # six checks driving real Chromium; skips cleanly without playwright
```

No build, lint, or typecheck — it's one file. The test count serves as a staleness floor: if fewer than ~2150 tests pass, your base is stale. Raise this when it drifts more than a hundred or so behind — it sat at ~1400 while the suite reached 1915, so a base 500 tests old passed the check whose whole job was to fail it. That number is parsed out of this sentence and ratcheted by a test, so keep the phrasing.

**The floor has a second half, and it is not optional.** A floor living in the tree is structurally blind to its own checkout being stale: a base 500 tests old carries the stale floor with it, compares 1500 against 1400, and passes. Only a number from *outside* the tree can see that, which is what the `staleness-floor` CI job does — it compares this branch's count against `origin/main`. If you touch one half, check the other.

## Architecture at a glance

1. **`node.text` is plain text, always.** Pills are opaque `[[type:key]]` tokens in text + records in sidecar arrays. Never store HTML in `node.text`.
2. **Two engines compose everything.** Engine 1 (`{…}`) generates: dice, grammar, markov, oracles, sequences. Engine 2 (`evalMath`) evaluates expressions. New capabilities go inside these — never new syntax.
3. **Markdown-first rendering.** `mdToHtml` is a per-line block parser. `node.type` for headings/quotes/to-dos is a derived hint from the text, not the renderer.
4. **Pure cores are DOM-free.** `evalMath`, `parseDice`, `runGrammar`, etc. return `null` on invalid input. Keep them testable in plain Node.

## Conventions

- **`markDirty()` is the single invalidation point.** Bumps `_varsVer`. All cross-node caches key on it. Every cache MUST register in `DOC_CACHES` at its declaration site (a `// doc-cache: <name>` marker + a `regDocCache` entry; the marker-parity test enforces the pair). New whole-tree caches go through `makeDocCache`, which registers and auto-tests them; skipping the registry means stale data AND a red CI.
- **Pure cores return `null` on invalid input.** Callers branch on `null`.
- **Custom OPML attributes are `_underscorePrefixed`.** Add serialize + parse in the same change or data drops on save.
- **Theme via CSS custom properties.** The palette lives in two homes: CSS `:root` AND `applyTheme`/`applyAccentCSS`. A CSS-only edit silently regresses when the user toggles theme.
- **No line numbers in docs.** Grep for function names — line numbers drift with every edit.

## UX principles

Laws copied **verbatim** from `guidance/ux-discipline.md` §2, which is canonical; a test pins them
equal. Do not paraphrase here — this is the always-loaded file, so a tightened restatement becomes
the rule everyone actually applies. That already happened: this table read "never syntax-only",
dropping the standard's "**at the floor**", which banned outright what the standard permits above
the floor (`ux.md`'s verbosity dial).

| # | Principle | Law |
|---|---|---|
| P1 | Predictable | A key, gesture, or word means the same thing everywhere. |
| P2 | Discoverable | Every capability has a visible front door — never syntax-only at the floor. |
| P3 | Reachable | Every interactive element is operable and announced to assistive tech. |
| P4 | Responsive | No silent success, no silent failure. |
| P5 | Coherent | One authoring language — reuse the existing syntax, don't mint a new one. |

**P1 and P5 win on conflict.** P3-3 (accessibility): keyboard is added *alongside* `mousedown`+`preventDefault`, never by converting to `click`/`<button>` — the caret invariant.

User-facing copy says **"point"** and **"pill"** (code keeps `node`/`artifact`). The hierarchy is **folder > document > point**. No em dashes in user strings — use AP punctuation only.

## Process — read this before you touch anything

| If your task is... | Your process |
|---|---|
| **UI change (any)** | Read `guidance/ux-discipline.md`. Read `guidance/design-language.md`. Clear `guidance/ux-definition-of-done.md`. Emit Conformance Statement (P1–P5 ✅/N/A). PR body starts "UX Conformance". |
| **Visual change only** | Same as UI + check the dual-home palette invariant: CSS `:root` AND `applyTheme`/`applyAccentCSS`. |
| **Adding a pill / artifact** | Read `guidance/adding-an-artifact.md` (12-step recipe + step-13 gate). P5 guard: new `{…}` branch or `evalMath` primitive first — never a new artifact unless it genuinely can't compose. |
| **Adding a user-facing feature** | Write concept-guide entry in same change (`guidance/concept-guide.md`). DRIFT GUARD: add `/` or `@` cmd ids to some entry's `covers:[…]`, or the tests fail. Freshen `guide/features.md`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zntznt/pointliner](https://github.com/zntznt/pointliner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
