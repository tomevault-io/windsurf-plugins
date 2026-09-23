---
trigger: always_on
description: Astryx is a public design-system repository. Never commit internal links,
---

# Astryx repository guidance

Astryx is a public design-system repository. Never commit internal links,
identifiers, service names, private operational instructions, or other
Meta-only context.

## Instruction surface

This `AGENTS.md` is the canonical, tool-agnostic instruction surface for the
repository. Add or change shared agent guidance here rather than duplicating it
in tool-specific instruction files. Put genuinely path-specific guidance in a
nested `AGENTS.md`.

## Start here

- Product builders: use `astryx docs`, component `{Name}.doc.mjs` files, and
  `packages/cli/assets/docs/`.
- Contributors: read `CONTRIBUTING.md` and the relevant guidance linked from
  `docs/README.md`.
- Pull requests: choose one primary intent and use its template under
  `.github/PULL_REQUEST_TEMPLATE/`; read `docs/contributing/pull-requests.md`
  before opening or reviewing a mixed change.
- Component work: derive the review's semantic triggers, load matching `current`
  global baseline claims with `node scripts/review-global-baselines.mjs
--authority-commit <base-sha> --review-head <head-sha> --triggers
<comma-separated-triggers>`, then read
  the component's `{Name}.spec.md` when one exists and any `module:*` records it
  lists. Load global records before narrower owners, but resolve the direct
  component or family owner first when it governs the exact delta. A global
  route exposes only the listed claim; it never makes the whole record govern
  the component or change. Preserve each matched record, claim, trigger, and
  match reason in the review receipt.
- Cross-component work: read the relevant contract under `docs/families/`,
  applicable design spec under `docs/design/`, and current architecture under
  `docs/architecture/`.
- Consequential shared-system changes: use a record under `docs/specs/`.

## Authority

Knowledge records declare `authority`:

- `draft`: not authoritative; may still need evidence or owner review;
- `current`: explicitly approved and authoritative;
- `archived`: context only, with a reason such as `superseded`, `withdrawn`,
  or `historical` and a replacement link when one exists.

Only `current` records govern implementation and review. Never infer approval
from merged code, silence, an old review, or an existing wiki page.

## Judgment boundary

Resolve checkable behavior from code, tests, and browser evidence. Ask a human
only when a stable public API, theme contract, ownership boundary, compatibility
policy, or genuinely subjective visual direction remains undecided. Ask one
question at a time.

Before drafting, reviewing, or implementing a proposed outcome, search current
records and open pull requests using the proposed canonical owner/id, affected
paths and exported symbols, and the behavior's semantic terms. Extend or project
the existing canonical owner by default. Create a new record only for a distinct
fact boundary, and state why the existing owner cannot contain it. Do not create
new policy for work that is already complete, owned, or superseded.

## Validation

Run `pnpm check:knowledge` after editing knowledge records or templates. A
material template-shape change requires a schema-version bump and migration of
active records; changing template guidance alone does not rewrite accepted
history.

## Custom Commands

### `/vibe-test [count]` - Run vibeability tests

Tests how well AGENTS.md helps LLMs generate correct Astryx component code.

**Usage:**

```
/vibe-test 5                    # Run 5 stratified sample tests (one-shot)
/vibe-test                      # Run all 21 tests (one-shot)
/vibe-test 5 --degradation      # Run 5 tests with degradation curve (10-turn)
```

**How to execute:**

1. Run `pnpm -F @astryxdesign/vibe-tests interactive --sample <count>` to set up iteration
2. Spawn parallel subagents (one per test prompt) to:
   - Read the task file from `results/<iteration>/tasks/{promptId}.json`
   - Generate code for the prompt using Astryx components (AGENTS.md auto-injected)
   - Self-evaluate for success/escape hatches
   - Write `.tsx` result to `results/<iteration>/results/{promptId}.tsx`
   - Write `.json` metadata to `results/<iteration>/results/{promptId}.json`
3. Trigger `gh workflow run vibe-screenshots.yml` to build previews and capture screenshots
4. Run `pnpm -F @astryxdesign/vibe-tests aggregate --iteration <id>` to see results

**Degradation mode (--degradation):**
Tests context retention across 10-turn conversations with filler, distractor, and recovery turns.
Probes at turns 0, 6, 8, 10 to measure quality degradation. Results show a line graph of each test's progression.

**Result format:**

```json
{
  "id": "<iter>-<promptId>",
  "timestamp": "...",
  "model": "claude-code-interactive",
  "persona": "naive",
  "promptCategory": "...",
  "trajectoryDepth": 0,
  "prompt": "...",
  "response": "<code>",
  "evaluation": {"success": true, "componentsUsed": [...], "escapeHatches": [...]}
}
```

Runners may also write an optional `<promptId>.provenance.json` sidecar beside the result metadata. The versioned, executor-neutral contract and fallback behavior are documented in `internal/vibe-tests/docs/execution-provenance.md`.

## AI Context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/astryx](https://github.com/facebook/astryx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
