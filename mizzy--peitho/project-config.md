---
trigger: always_on
description: An HTML-native presentation tool with Markdown as the source of truth. The authoritative design reference is `docs/PEITHO_KICKOFF.md` (the kickoff spec). When in doubt about a design decision, check §18 "Undecided Items" in the spec; for new decisions not covered there, check with the author.
---

# peitho

An HTML-native presentation tool with Markdown as the source of truth. The authoritative design reference is `docs/PEITHO_KICKOFF.md` (the kickoff spec). When in doubt about a design decision, check §18 "Undecided Items" in the spec; for new decisions not covered there, check with the author.

## The three pillars (invariants that must not be broken)

1. **Separation of content and design**: Content is Markdown, design is layout HTML+CSS. Do not mix them
2. **Git-manageable HTML/CSS layouts**: The layout itself is the schema (`<slot name accepts arity>`). Do not split the contract into a separate file
3. **Type-checked slot contracts and keyed overrides**: Slot excess/deficiency, type mismatches, broken references, and unassigned content are all build errors with line numbers + help. **Silent dropping is absolutely forbidden** (never let the parser swallow unknown structures with `_ => {}`)

Other invariants:
- typestate `Parsed→Mapped→Checked→Rendered`. Phase constructors are private within the crate. An unchecked deck cannot be passed to the renderer (pinned down by a compile_fail doctest)
- Multiple layouts use hybrid dispatch (type-driven approach from §18 adopted at the author's discretion, 2026-07-03): explicit `{"layout":"name"}` > unconditional if there's only one > unique structural match. Ambiguous or zero matches are never silently resolved — they are build errors. Slides carry their own Layout from Mapped onward, so no lookup-failure path is created downstream
- Syntax highlighting is done at build time with syntect (adopted 2026-07-03). Output is spans with `hl-*` classes, colors come from the theme CSS. Unknown language tags are a parse-time error with a line number (no tag means plain text)
- Deck-level settings ride in YAML frontmatter at the top of the deck (author decision 2026-07-03: "zero-config, settings live in Markdown frontmatter"). First key: `time` (planned presentation time, `15m`/`90s`/`1h30m`/bare integer minutes). The frontmatter body is restricted to flat `key:` lines (trailing style blank lines allowed); unknown keys, invalid values, markdown swallowed by a missing closing `---`, a leading `---` without valid frontmatter, and settings anywhere but the top are all line-numbered build errors with help — no silent path exists. Validation happens once at `PlannedTime`'s construction (nonzero, overflow, ≤ `Number.MAX_SAFE_INTEGER` so JSON.parse can't round it), and the validated newtype rides on `Deck<P>` through every phase; milliseconds appear only at consumption boundaries (manifest). Design record: `docs/specs/2026-07-03-time-tracking-design.md`. Migrating `--layouts`/`--css` into frontmatter is Issue #62
- Agenda sections (2026-07-04): a slide's page settings comment may declare `{"section":"Name","time":"1m"}` (section and time must appear together; the marked slide starts a section that runs until the next marker). Ranges are derived at parse end; if any marker exists the first slide must carry one; when frontmatter `time` is present the section total must equal it, when absent the total becomes the deck's planned time (so a second validation point exists at parse end, not only at `PlannedTime` construction). A slide accepts at most ONE page settings comment — a second one is a build error (the old field-wise last-wins merge could silently drop a section marker). Sections ride `DeckSettings` through every phase into `manifest.json` `sections` (always present, empty when unused); the presenter agenda measures actuals client-side (cumulative, in-memory). Design record: `docs/specs/2026-07-04-presenter-agenda-design.md`
- Speaker notes (2026-07-04): non-JSON HTML comments in a slide body are collected as that slide's speaker note (Marp / k1LoW/deck-style). JSON-vs-plaintext is the discriminator (reuses `parse_page_comment`) so page settings and notes coexist; empty comments are ignored; multiple comments per slide are joined with a blank line; position is unconstrained (before/inside/after the body). Notes ride `notes: Option<String>` on the slide through every phase and are emitted into `notes.json` (keyed by `SlideKey`, built at Rendered when keys are final). **Notes never enter `dist/`** — only `peitho present` reads `notes.json`; the publish contamination check enforces this. Presenter renders notes as `textContent` (plaintext v1); Markdown interpretation is a future extension the versioned schema leaves room for. Design record: `docs/plans/2026-07-04-speaker-notes.md`
- Presenter view (2026-07-04, PR #79 + follow-ups): state-synced timer UI with a fixed 42vh notes panel and a height-derived 16:9 preview stage that keeps left-column widths stable. Space bar maps to `playpause` (start/pause/resume derived from `TimerState`); the presenter emits `peitho:timercontrol` and the shell executes the transition. Empty notes render as a dimmed placeholder rather than a blank block. Design records: `docs/plans/2026-07-04-presenter-redesign.md`, `docs/plans/2026-07-04-presenter-redesign-followups.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mizzy/peitho](https://github.com/mizzy/peitho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
