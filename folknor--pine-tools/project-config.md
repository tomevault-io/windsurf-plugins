---
trigger: always_on
description: **This project uses pnpm.** Do not use npm or yarn.
---

# pine-tools

**This project uses pnpm.** Do not use npm or yarn.

```bash
pnpm install          # Install dependencies
pnpm run <script>     # Run scripts
pnpm test             # Run tests
pnpm check            # Formatter + linter + assist (biome). Alias: pnpm lint
```

---

## Rules

### General rules

- Don't use gremlins! Em-dash, en-dash, strange quotes, whatever - they're
  all verboten.
- No emojis in docs (notes, gotchas, READMEs, TODO, commit messages, etc.).
  Plain text only - use a word like "WARNING"/"NOTE" instead of a symbol.
- Don't remind the user of the rules. They wrote them, so they know them.
- The user can exempt you from any rule at any time.
- Prefer structural or semantic criteria over arbitrary numeric thresholds
  in heuristics. Magic numbers (`> 10x`, `count > N`, `> N lines`) are
  brittle and hide the real criterion. If a number only surfaces candidates
  to look at, say so and treat it as exploratory, not a rule.
- Once a plan is agreed, execute it end-to-end without pausing for per-step
  go-aheads. Surface genuine new decision points (an unexpected regression,
  a fork the plan didn't cover), but don't re-confirm already-agreed steps.
  (Commits stay the exception - see git commit rules.)

### Doc scope (where things go)

- `notes/todo.md` is **pending work only** - things someone could pick up.
  Resolved/reverted items, past investigations, and indexes of completed
  work do NOT belong; those live in git log, `investigations/`, and
  `gotchas/`. Link to the canonical home instead of copying. (It lived at
  `TODO.md` in the repository root until 2026-08-25; older investigation
  notes refer to it by that name.)
- `investigations/INV###` is **only** for lint/parser/lexer disagreements
  that come with a minimal `.pine` repro. Pipeline/data/scraper/architecture
  work goes in `notes/todo.md`; unfixable TV or Pine-language quirks go in
  `gotchas/`.

---

## Methodology - we aim to be MORE correct than TradingView's pine-lint

TradingView's `pine-lint` is a reference, not the spec. It has real bugs:
it stops at the first error, blames whitespace for a missing `)`
elsewhere in the file, silently accepts nonsense expressions, and its
results sometimes change run-to-run for no apparent reason. **Matching
it is not the goal.** Our linter should catch what TV catches *and*
what TV misses.

### Hard rules

- **TV silence is evidence, not authority.** When TV is silent and we
  flag an expression, that is a disagreement - it might be us being
  wrong, or it might be us correctly catching something TV missed.
  Investigate the expression itself before deciding.
- **Never relax a check just because TV is silent.** If the existing
  checker is stricter than TV, treat the comment / commit that
  introduced it as a signal that someone already weighed this trade-off.
- **Disagreements are claims, not bugs.** The "false positive" /
  "false negative" labels in `lint-reports/real-failures.json` are
  position-based heuristics. Treat them as "things to look at," not
  "things to fix."

### Per-disagreement workflow

For every concrete TV-vs-us discrepancy we choose to act on:

1. **Reproduce** with a minimal `.pine` fixture in
   `packages/core/test/fixtures/regression/`. The discovery test runner
   picks it up automatically - a repro that doesn't fail-on-regression
   is just a paragraph with code in it. Prefer the
   `// @expects error: line=N, message="..."` directive form; the
   count forms `// @expects errors: N` / `warnings: N` are also
   supported. Use `// @expects ast: body.0.type = "IfStatement"`
   for targeted AST-shape locks when diagnostics alone could pass with the
   wrong parse shape. AST directives support dot paths with numeric array
   indexes, `=`, `~= /regex/`, and `exists`. Warnings assert the
   SemanticAnalyzer channel (same as the CLI); validator warnings are stripped.

   **`line=N` counts the fixture WITHOUT its `// @` directive lines.** The
   harness consumes every leading `// @...` line and validates what remains,
   so a fixture with seven directive lines reports an error the CLI puts at
   line 14 as line 7. Ordinary `//` comments and blank lines are KEPT and do
   count. Easiest not to compute it: run `pine-lint -H <fixture>`, then
   subtract the directive-line count from each line it reports. A wrong `N`
   fails as `Expected error not found` while the `errors: N` count still
   passes, which reads like a message-match failure and is not one.

   **A `parse: fail` fixture cannot assert anything about validation.** The
   harness runs the validator only when there are zero parse errors, so on
   such a fixture `// @expects errors: N` is vacuous - it can never go red,
   whatever N is. The CLI does NOT behave this way: it validates regardless
   and will happily emit an undefined-variable cascade behind a parse error.
   So a fix whose point is suppressing that cascade cannot be pinned by a
   fixture at all; pin the parse errors there and carry the validation half
   in a probe with the expected `pine-lint -H` output written down.
2. **Open an investigation** under `investigations/INV###-name/` with
   `notes.md` and the repro file (or a pointer to the regression
   fixture). Sequential numbering, never reuse. Index entries live in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [folknor/pine-tools](https://github.com/folknor/pine-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
