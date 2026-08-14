---
trigger: always_on
description: [argot](https://argot.tmonier.com) is a guardrail that flags code **foreign to a
---

# Working with argot

[argot](https://argot.tmonier.com) is a guardrail that flags code **foreign to a
repo's own patterns** — learned from its git history. Five learned detectors:
a foreign dependency/API/idiom, a function the repo already has, code filed in
the wrong place, an internal import that reverses the repo's layering, and a
test weakened, disabled, or deleted alongside the production change it covers
— plus repo-local **custom rules** the team scripts itself (see below). It
answers *"is this how we write things here?"*, not *"is this valid?"*. This
file tells a coding agent how to use it well. (Developing argot itself? See
`CLAUDE.md`.)

## The contract: surface, don't enforce

argot is a **probabilistic** linter. It has false positives, and its confidence
depends on the repo. **A hit is a prompt to think, never a reason to refuse.**

- Never block a commit, fail a task, or refuse to proceed because argot fired.
- Never auto-"fix" a hit by rewriting code the human didn't ask you to change.
- The human always has the last word. Your job is to *surface* divergences and
  *record decisions*, not to enforce.

## Two moments

1. **Setup (once per repo).** Fit the voice model and decide what shouldn't
   shape it. → `argot init` (also builds the semantic index — the embedding
   model ships inside the binary, so this needs no network). Review and commit
   `argot.toml` **and the generated `.argot/` fit snapshot**; CI consumes that
   snapshot and never fits. Then use the **argot-setup** skill for anything with
   generated/vendored/data directories. See
   [Setup](https://argot.tmonier.com/docs/setup/).
2. **Check (per change).** Before committing code you generated or edited, score
   it. → `argot check`, or the **argot-check** skill.

Snapshot maintenance is adaptive, not scheduled: `argot status` compares the
accepted tree with the fitted source/function/layout profile. Commit count and age are
context only unless the team explicitly configures `[fit] refresh-after` as a
backstop. `watch` is informational; only `recommended` or
`strongly_recommended` should prompt maintenance. Use the **argot-refresh**
skill: `refresh.next_action` says whether to fit directly or review scope first;
the skill also reviews mutes before the local fit, review, and commit. No
command or CI workflow refits automatically.

## Reading `argot check`

Run `argot check --format json` for machine output. Each hit carries a `rule`,
a `severity` (`error`/`warn` — whether it fails the check), a `confidence` tier
(`unusual`/`suspicious`/`foreign` — how strong the evidence is, display only),
an evidence trail, and a stable `hash`. **Branch on the rule, not the
confidence tier** — `redundant` is pinned to `unusual` confidence and is still
one of the most actionable findings argot makes:

| Rule | Group | It means | What to do |
|---|---|---|---|
| `foreign-import` | voice | an import of a dependency the repo has never used | Check the evidence line for what the repo uses instead; prefer the in-voice option unless the new dependency is deliberate. |
| `unfamiliar-callee` | voice | a call to something this kind of file never calls | Same: compare against the named common callees. |
| `rare-tokens` | voice | a token sequence statistically foreign to the repo's voice | Read the flagged identifiers; rewrite with the repo's vocabulary if unintended. |
| `convention` | voice | a construction that breaks a learned repo convention | As above. |
| `superseded` | voice | new code uses a pattern this repo has replaced — mined from history, or declared in `argot.toml` | Use the replacement named in the evidence (the commits that made the switch, or the declared reason). |
| `redundant` | semantic | this new function duplicates one the repo already has | **Open the file the evidence names** (`↳ duplicates X (path:line)`), compare, and use the existing function instead — or justify and mute. |
| `misplaced` | semantic | this function's nearest kin all live in another area | Propose moving it to the named area, or justify its placement. |
| `layering` | architecture | this internal import reverses the repo's layer direction | Don't introduce the import — invert the dependency or go through the intended layer. |
| `test-deleted` | integrity | a test removed while the production code it exercised still exists | Restore the test or explain why it's obsolete; if the deletion is legitimate (feature removed), the code that exercised it should be gone too. |
| `test-disabled` | integrity | a skip/ignore marker added, or a test gutted, while production code changes | Un-skip and fix the code, or record why the skip is temporary; skipping to make a failing suite green is the exact behavior this rule exists to catch. |
| `test-weakened` | integrity | assertions removed, tautologized, or loosened while production code changes | Restore the assertion strength; if the expected value legitimately changed, say why in the commit/PR rather than silently retargeting. |

Rules are configurable like any linter: `argot rules` lists them; `argot.toml
[rules]` or `argot check --rule <name|group>=<error|warn|off>` sets severities.
Everything defaults to `error` except `test-weakened` and `superseded`, which

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [get-tmonier/argot](https://github.com/get-tmonier/argot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
