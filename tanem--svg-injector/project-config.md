---
trigger: always_on
description: Rules for coding agents that the code and config don't already state. Keep it
---

# AGENTS.md

Rules for coding agents that the code and config don't already state. Keep it
that way: a constraint that can live in a comment next to the thing it
constrains belongs there, not here.

## Writing

- NZ English everywhere ("colour", "behaviour", "initialise").
- Match a document's length to what it needs. Cover the substance, then
  stop: no filler sections, restated summaries or boilerplate.
- Simple, direct technical language. No marketing speak.
- Commit subjects are one capitalised line, `git log --oneline` style. Add a
  body whenever the change had a reason the diff does not show: what it fixes,
  what it rules out, what constraint forced the shape it has. Mechanical
  changes need none.
- No conventional-commit prefixes (`feat:`, `fix:`, `chore(deps):`) in commit
  subjects or PR titles. Write a plain capitalised sentence. Nothing reads the
  prefix: the version bump comes from the PR label, and renovate is set to
  `semanticCommits: "disabled"` to match.
- PR titles are copied verbatim into the generated release notes, so write them
  as the changelog line you want readers to see.
- Hard-wrap commit message bodies at 72 columns; `git log` does not reflow
  them. Do not hard-wrap PR or issue descriptions: GitHub reflows markdown,
  and its web editor leaves wrapped source ragged once anyone edits it.
- `README.md` follows
  [standard-readme](https://github.com/RichardLitt/standard-readme). Per-feature
  detail belongs in `examples/*/README.md`, linked from there.
- `README.md` states current behaviour; `MIGRATION.md` states what changed and
  keeps the upgrade steps in full. Neither re-derives the other.
- Code comments record non-obvious behaviour, constraints and decisions, not
  what the line already says.

## Architecture

`SVGInjector` normalises its argument, then runs one pipeline per element:
split the sprite fragment off the URL, load, transform, swap.

- `svg-injector.ts` owns the `afterEach` and `afterAll` accounting. Every path
  out of `injectElement`, errors included, calls back exactly once. Add one
  that doesn't and `afterAll` silently never fires.
- `defer.ts` enforces the callback timing `svg-injector.ts` documents. A path
  already running inside an XHR event or a deferred task calls back directly,
  which is why `load-svg-uncached.ts` and the `handleLoadedSvg` error paths
  hold no `defer`.
- `inject-element.ts` is that per-element pipeline, and the only module that
  chooses a load path.
- `parse-data-url.ts` intercepts `data:image/svg+xml` before any request is
  made, so data URLs never reach the XHR layer.
- `load-svg-cached.ts` and `load-svg-uncached.ts` wrap `make-ajax-request.ts`.
  The cache key is the URL with the fragment stripped, so every symbol taken
  from one sprite shares a single request.
- `extract-symbol.ts`, `renumerate-svg-iri-elements.ts` and
  `eval-svg-scripts.ts` are the transform steps, applied in that order.

XHR is a decision, not leftover legacy. It carries the `file://` allowances
`make-ajax-request.ts` documents, and its content-type check runs at
`readyState` 2 so a rejected response aborts before its body arrives. fetch
does neither. Don't migrate it without new evidence. Safari is the engine that
decides anything `file://` and the only one those allowances are load-bearing
in; measured on Chrome 151, Safari 26.5 and Firefox 146.

## Known limitations

The consumer-facing ones are in `examples/sprite-usage/README.md`,
`examples/data-url-usage/README.md` and the `renumerateIRIElements` section of
`README.md` (background on that one:
[#14 (comment)](https://github.com/tanem/svg-injector/issues/14#issuecomment-457270023)).
These are the ones stated nowhere else, and they shape what can be built on
top.

### SVG sprites

- The fragment is matched verbatim against the symbol id, so a percent-encoded
  fragment (`sprite.svg#caf%C3%A9`) never matches the decoded id and fails with
  `Symbol "caf%C3%A9" not found in ...`. Browsers decode the fragment for a
  native `<use>`, so this diverges from platform behaviour. The literal form
  (`sprite.svg#café`) works.

### Data URLs

- The scheme and media type are matched case-sensitively, though RFC 2397 makes
  both case-insensitive, so `data:IMAGE/SVG+XML,...` falls through to XHR.
  Browsers fetch such a URL over XHR without complaint, so the only exposure is
  a context where a strict CSP blocks the request.
- `DOMParser` error detection is best effort: browsers embed a `<parsererror>`
  element rather than throwing, and its message format varies by browser.

### IRI renumeration

- String references inside `<script>` blocks are not updated, so
  `document.getElementById('oldId')` keeps pointing at the old ID.
- CSS ID selectors in `<style>` elements are not updated. Only `url(#id)`
  references within the style text are rewritten, so a rule like
  `#myId { fill: red }` still references the old ID.

## Build & test

`npm run test:playwright` is the development loop, run against a current
`npm run build`: the suite loads the built IIFE bundle, so an unbuilt source
change is not under test. `npm test` is the full gate, and also builds and
verifies every example; `npm run test:examples` is the shorter loop for a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanem/svg-injector](https://github.com/tanem/svg-injector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
