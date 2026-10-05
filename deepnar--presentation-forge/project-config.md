---
trigger: always_on
description: Generator for academic presentations and reports. Node + pptxgenjs. The product
---

# Presentation Forge — working agreement

Generator for academic presentations and reports. Node + pptxgenjs. The product
is **hosted**, and the model behind it is the TCET CoE gateway (the "Auto"
tier) — see *Which model to judge the product by*, below. Local Ollama still
works and is still supported; it is no longer the thing being aimed at.

## The one rule that governs everything

**The model never writes layout code.** Three layers, strictly separated:

| Layer | Owns | Written by |
|---|---|---|
| chrome | banner, crest, presenter, slide number | `src/chrome.js`, locked |
| theme | palette, type, spacing, shape | `themes/*.yaml`, human |
| content | what the slides say | the model, as validated YAML |

A change that lets content specify a coordinate, a hex colour, or a font name is
wrong regardless of how convenient it looks. If a slide needs a layout that does
not exist, add a slide *type* — do not add an escape hatch to the schema.

Themes are split into `tokens` (exact machine values, read only by the renderer)
and `voice` (tone guidance, read only by the model). These must never overlap:
the moment a model can see a hex value it will start inventing them.

## Repository layout

```
src/            renderer, validator, fitter, chrome, preview      (CLI + API share this)
themes/         one YAML per design language
schema/         deck.schema.json — the content contract
config/         identity.yaml — remembered defaults, user-editable
brand/logos/    raw source marks (any format)
brand/generated/ normalised marks — produced, never hand-edited
app/server/     Express; a transport over src/, holds no logic
app/web/        Vite + React UI
decks/<slug>/   deck.yaml, meta.yaml, out/
reference/      the institutional templates output is checked against
docs/           ROADMAP.md (all work), PRODUCTION, ECONOMICS, ARCHITECTURE, TRAPS
```

`app/server` must stay a thin wrapper. Anything it can do, the CLI must also do,
because the pipeline has to run headless.

## Commit discipline

1. **Impersonal, factual messages.** Describe what changed and why, in the
   repository's voice. Never narrate the session — no "the user said…",
   "as requested…", "we decided…", "per our discussion". Subject ≤ ~70 chars;
   the body explains the *why*, not the *what* (the diff shows the what).
2. **Many small, focused commits.** Split by concern, not by session — one
   logical change per commit (renderer / theme / docs / tests as separate
   commits). Never one massive end-of-session commit. If a message needs bullets
   to list unrelated changes, it should have been several commits.
3. **Stage in logical chunks** — `git add <specific paths>` per commit, never
   `git add -A` once.
4. **No AI attribution.** No `Co-Authored-By` trailers, no "generated with", no
   tool names anywhere in commit messages or code comments.

## Working the roadmap

`docs/ROADMAP.md` is both the plan and the progress tracker. It is not a spec.

**It is also the ONLY place future work is written down.** The moment a piece
of work is discussed — a defect noticed in passing, a feature the user asks
about, a direction agreed in conversation, an idea that came out of building
something else — it gets an entry in `docs/ROADMAP.md` in that same session,
before the session ends. Not in a handoff, not in a section of another doc, not
only in the conversation.

This is a rule because it has already gone wrong: work items accumulated in
`PRODUCTION.md` §4 and §5 in parallel with the roadmap, so there were two
lists, and whichever was not being edited went stale. The other docs have their
own jobs and none of them is holding work:

| file | answers |
|---|---|
| `docs/ROADMAP.md` | **what is to be done, what was done, and why** |
| `docs/PRODUCTION.md` | what stops real users — launch readiness only |
| `docs/ECONOMICS.md` | what it costs and what it can be sold for |
| `docs/ARCHITECTURE.md` | how the system is built, as built |
| `docs/TRAPS.md` | what has bitten before, cross-cutting |
| `docs/BLOCKED.md` | what cannot be worked on, and who unblocks it |
| `docs/HANDOFF.md` | the current state, for the next session; overwritten |

When one of those files needs to mention future work, it LINKS to the roadmap
entry rather than restating it. A pointer that goes stale is visible; a second
copy that goes stale is not.

**Keep it current, not just appended to.** An entry whose reality has changed
is edited when it changes — including entries nobody is working on, when a
neighbouring change makes them wrong. A roadmap that is only ever added to
becomes an archive, and an archive is not a plan.

- **Entries are intent + rationale, not specs.** When an item's turn comes,
  discuss the concrete implementation before writing code. Never build straight
  from the entry text.
- **Stop and ask when the item needs direction — do not substitute another.**
  Some items are taste and product judgement rather than defects: the front end
  and the landing page, anything that changes what a surface IS rather than
  whether it works, and any item whose entry says to agree direction first. On
  those, say what the options are and WAIT for an answer. Do not assume the
  human is away and quietly pick a different item instead; that is how the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deepnar/presentation-forge](https://github.com/Deepnar/presentation-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
