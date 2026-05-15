---
trigger: always_on
description: This file captures only what cannot be inferred from the codebase itself.
---

# CLAUDE.md

This file captures only what cannot be inferred from the codebase itself.

## Rules for editing this file

Both developers and AI agents are expected to add entries as they encounter surprises.

- **Add an entry** when you encounter something unexpected: a build quirk, a non-obvious constraint, a dependency gotcha, or any behavior that would surprise the next agent or developer.
- **Add an entry** when a developer flags an anti-pattern produced by AI — describe the anti-pattern and the preferred alternative.
- **Do not** add codebase overviews, directory listings, or anything discoverable by reading the source.
- Keep entries concise: one line per lesson, grouped under a heading if a theme emerges.

## Streaming divergences

The parser intentionally diverges from GFM in well-defined places where
spec-correct behavior would force buffering past the next emitted event,
breaking the typewriter-style streaming UX. **These are not bugs — read this
section before "fixing" a DIVERGENCE-marked test.** Tests covering these
cases are explicitly named with `DIVERGENCE` in their function name.

### Core invariant

The semantic event stream is **append-only**. Once a `text` / `mark` / `unmark`
event is emitted to the downstream collector, it cannot be retracted or
modified. All divergences below trace back to this constraint.

### What we DO buffer (bounded, well-defined cases)

The parser does buffer in a few specific places — never past a single inline
construct or two-line lookahead, never past the next emitted event:

- **N≥2 inline code spans** (`` `` …content… `` ``): full content held until
  the close-run resolves the GFM strip rule.
- **GFM tables** (`BlockMode.TableHeaderPending`): a `|`-line is held for one
  more line to see if a separator confirms the table.
- **List blocks**: the entire list block is buffered before structural emit.
- **Inline link/image labels** (`[label]` / `![alt]`): a `RedirectingCollector`
  capture holds events between `[` and `](url)` (or abort) so `<a>`/`<img>`
  can wrap the rendered content (Phase 3a).
- **HTML 6/7 blocks**: rich state machine with sub-parse mode for nested
  Markdown — but no event retraction.
- **Extended autolinks (GFM §6.9)**: the `AutolinkCollector` wrapper holds
  every `text` event arriving inside an inline context until the next
  `mark`/`unmark` (or end-of-flow). At drain time the buffered text is scanned
  for `www.` / `http(s)://` / `ftp://` / `mailto:` / `xmpp:` / bare email
  patterns; non-matches replay the original events untouched, matches replay
  with `<a>` woven in. This delays paragraph text emission until the next
  inline mark/unmark — a deliberate trade-off vs. correctness for the
  email-autolink case (we can't recognise `local-part@host` until `@` is seen,
  by which point the local part has already been emitted otherwise).

### What we DO NOT do (the divergences)

#### Inline state cannot span line breaks

`flushInline` force-closes every inline state (`code`, `math`, `strong`, `em`,
`del`, `mark`, `sup`) at every line/block boundary. Consequence: multi-line
inline code spans, multi-line emphasis runs, etc. close at the line break.
Lifting this would require inline state to persist past `\n` and the
paragraph/continuation handlers to suppress structural `\n` events while inside
an open inline span.

**Affected**: GFM ex 357/358/359 (unmatched code-span openers), 340/341/350
(N=1 strip rule), some §6.13 soft-break edge cases.

#### Eager open commits for code, math, em (no retroactive close)

Inline code, math, and emphasis emit their `mark` event on open detection —
content streams immediately. If the close never arrives (or the source's
structure wants the open delimiters as literal), we cannot retract. `<code>` at
end of a code-spanless paragraph closes via `flushInline` rather than replaying
the opener as text.

Trade-off was made in favor of typewriter UX — code/em should appear as you
type, not pop in only after the closer.

#### Lists are always rendered loose

`<p>` wraps every list-item's first content. Tight/loose can only be decided
after the entire list closes (a blank line in the very last item retroactively
makes prior items loose). Once `<li>foo` text is emitted, it cannot be wrapped
in `<p>` after-the-fact. Buffering the whole list breaks streaming.

CSS picks up the slack: `li > p:only-child { margin: 0 }` collapses the visual
gap so loose-rendered tight lists look right.

#### Emphasis resolution is flanking-aware but not delimiter-stack-aware

`resolveEmphasisRun` consults `inlineOpenStack` for em/strong frames already
open and uses CommonMark flanking (rules 1–8). When the run can legitimately
close, `closeInlineDownTo` walks the stack LIFO closing inner frames first.

But: the spec's full process_emphasis algorithm needs a paragraph-wide delimiter
stack with bidirectional pairing — incompatible with our streaming, where
events are emitted as runs resolve. So `*foo**bar*` produces
`<em>foo<strong>bar</strong></em>` (we close strong inside em on the closer);
spec emits `<em>foo**bar</em>` because no `**` closer follows, but that
requires whole-paragraph buffering.

#### Tables only at block boundary, never interrupting paragraphs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xemantic/markanywhere](https://github.com/xemantic/markanywhere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
