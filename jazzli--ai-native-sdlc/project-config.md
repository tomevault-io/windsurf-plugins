---
trigger: always_on
description: Research content engine behind a public reference on AI-native software
---

# AGENTS.md

Research content engine behind a public reference on AI-native software
development, live at https://jazzli.github.io/ai-native-sdlc/. The content is
plain markdown built around one traceability chain:

`docs/playbook.md` → `docs/questions/*.md` → `sources.md#<citekey>` → primary source

Two supporting trees serve it: `site/` (Astro; publishes the content) and
`watch/` (the daily discovery sweep). Read `docs/protocol.md` (one page,
five rules) before changing content.

## Writing for the public record

Commits, pull requests, documentation, and site copy are read by people
deciding whether to trust the positions. Write them as durable engineering
artifacts, not as a record of how the work went.

- Commit subjects: sentence case, imperative, no trailing period, 72
  characters or fewer. One logical change per commit.
- Commit bodies only when the subject is insufficient. State the problem,
  the decision, and any consequence a future reader needs. Omit
  implementation chronology, dead ends, and self-assessment.
- Pull requests answer why the change exists, what materially changes, what
  a reviewer must decide, and how it was verified. A small change gets a
  short description. Do not narrate the process that produced it.
- Prose is direct and calibrated. Avoid first-person narration of the work,
  rhetorical emphasis, decorative formatting, and emoji.
- Preserve the distinctions the protocol depends on — evidence, inference,
  confidence, falsifiability — and state them plainly.
- American English throughout. Recurring terms are spelled: AI-native,
  AI-Native SDLC (the project), agent, coding agent, SDLC, GitHub, open
  source (noun) and open-source (adjective), machine-readable, fail closed
  (verb) and fail-closed (adjective), pre-commit, changelog.

### Worked examples

A subject alone, where the change explains itself:

```
Wrap changelog table cells at the measure
```

A body earns its place when a decision needs explaining:

```
Serve the digest as text, without breaking the published URL

GitHub Pages types responses by file extension and offers no override,
so /positions.digest was served as application/octet-stream. The .txt
form is now canonical. The original URL still serves the same bytes
because it was already published in the drift check, and this project
does not break a URL it has put in front of adopters.

Content is unchanged, so no adopter sees drift from this.
```

The same change written as a work log. Do not do this:

```
Fix the digest MIME type

First I tried setting the Content-Type header in the Astro route, but
that turned out not to matter because Pages serves from disk. Then I
looked for a _headers file, which GitHub Pages does not support.
Eventually I realised the only lever is the file extension, so I added
a .txt endpoint. I also had to fix a test that grabbed the wrong code
block.
```

Both describe one change. The second records why the extension is the
only lever available, and that adopters are unaffected — the two things
a reader will need later. The third records the author's afternoon.

### AI authorship

This project is built with coding agents, and says so once, in the README,
rather than in every artifact they touch. Do not add `Co-Authored-By:`
trailers naming a model, or tool-attribution footers to pull requests: the
model that happened to run is not authorship, and downstream readers cannot
act on it. The maintainer is accountable for everything merged here.

Automated actors that write to this repository — the discovery sweep, the
falsifier watch, Dependabot — identify themselves in the content they post.

## Requirements

- Never assert more than the cited registry entry records. If a claim needs
  more, enrich the entry from the primary source first (protocol Rule 5),
  then update the note, then the playbook — in that order (Rule 4).
- Unsourced reasoning must be labeled as inference in the text, not left
  reading as sourced.
- New `sources.md` entries: citekey anchor (`<a id="...">`, kebab-case,
  permanent), a **Filter:** line stating which of the three signal-filter
  questions it passes, and verification at the primary source on admission.
- Question notes: frontmatter is exactly `title`, `status`
  (`open | working-answer | parked`), `updated` (YYYY-MM-DD); a
  `## What would change my mind` section with concrete falsifiers is
  mandatory. Bump `updated` when content changes.
- Plain relative markdown links only; never `[[wikilinks]]`. The playbook
  never cites `sources.md` directly.
- Verify with `lychee --config lychee.toml --no-progress .` before pushing;
  never weaken `lychee.toml` or the workflow to get green. New accept-codes
  or excludes need a comment naming the host and reason.
- `watch/` runs the daily discovery sweep. Keep `discover.mjs` a thin I/O
  shell: pure logic belongs in `watch/lib.mjs`, which the site's vitest
  suite covers, and `npm run lint` / `format:check` in `site/` span both
  trees. Nothing load-bearing here sits outside a gate.
- `site/tests/rendered.test.ts` asserts invariants over built output and
  skips when `site/dist` is absent — run `npm run build` before `npm test`
  to exercise it locally, as CI does (`REQUIRE_RENDERED=1`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jazzli/ai-native-sdlc](https://github.com/jazzli/ai-native-sdlc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
