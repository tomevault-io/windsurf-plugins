---
trigger: always_on
description: Principles for anyone working in this repo, human or agent. `CONTEXT.md` is the
---

# localmd — working agreements

Principles for anyone working in this repo, human or agent. `CONTEXT.md` is the
glossary — read it before renaming or reinterpreting a concept ("index",
"session", "slot", "bundled", "connection", …). `CONTRIBUTING.md` covers the
mechanics of a pull request; this file is about the judgment calls.

This is the only copy. `CLAUDE.md` is a one-line import of it, because Claude
Code looks for that name while most agents read this one — add a pointer for
whatever tool you use, never a second copy to keep in step.

## Language

**This is an English repo.** Code, identifiers, comments, docs, commit messages
and LLM prompts are written in English by default — regardless of the language
the task was discussed in — unless explicitly asked otherwise. The only
intentional Chinese in the tree: the `zh` values in i18n catalogs
(`src/i18n/locales/*`), `LOCALE_NAMES`, and the CJK regex/fixture data in lib
(annotations / markdown / tts / pdf extraction).

## Product principles

- **The KB is a soft constraint.** Every convention — layout, frontmatter,
  linking — is a suggestion the agent follows and recommends, never a
  validation gate. Users hand-edit, move and delete files outside the app;
  nothing may break or nag when they do.
- **The user's structure wins.** The `raw/` + `wiki/` layout is only the
  scaffold offered for brand-new empty KBs. Opening an existing folder must
  never graft our layout onto it: automatic writes land in a neutral `inbox/`
  when the KB has no `raw/` tree, and the agent files content according to the
  user's own organization. AGENTS.md inside a knowledge base is where that
  KB's structure is described — offer to write one that documents *their*
  layout; don't impose ours.
- **The agent adds; it does not rearrange.** What stops someone from opening a
  folder they have let get messy is not "the AI will be wrong", it is "the AI
  will move something, or delete it". The default posture is therefore
  additive: the agent writes *new* files that point inward at untouched
  material — topic and entity pages, `log.md`, wikilinks, `[[1:b14-3]]` block
  citations — and at most proposes a reorganization, never performs one
  unasked. New files are the encouraged half of that posture, not a tolerated
  one: an index page gives the material spatial structure, `log.md` gives it a
  history — structure added *beside* the user's own, never carved into it. The
  line that may not be crossed is writing a *record*: a cache, a queue, any
  file whose correctness the machinery must maintain. A note survives being
  hand-edited, moved or deleted; a record left to a human starts to lie
  (`docs/llm-wiki-prior-art.md` rejected the navigation-cache layer and the
  review queue for exactly this, and only this). Tags have two permitted homes
  and one forbidden one: the frontmatter of pages the agent itself wrote (the
  default), or — as a batch the user approves, in one commit — the frontmatter
  of pages they wrote. Never a sidecar tag store only this app can read: tags
  no other tool can see are a database hidden inside someone's folder, and the
  whole promise is that leaving costs nothing.
  Today this is a rule we keep, not an invariant we can claim. What enforces it
  is review — the write snapshot and diff, ask-first mode, restorable text
  deletes, `.git` off limits, git underneath — while `delete_path` and
  `move_path` reach anything the user granted. Until a write guard makes it
  structural, copy may say the agent *asks*; it may not say the agent *cannot*.
  That gap is exactly what this reader is listening for.
- **Recall is a view or a note, never a record.** Surfacing an old note at the
  right moment must not write machine state into someone's folder — the
  reasoning that rejected nashsu's review queue and nvk's navigation-cache
  layer (`docs/llm-wiki-prior-art.md`). The computed signals already exist in
  `computeLint`, derived from `kbIndex` with no page reads:
  `unreferencedSources` (material sitting unread), `stalePages`,
  `staleLogEntries`, `weaklyLinked` — render them on demand, never persist
  them. A finding worth *keeping* does have a durable home: a dated `log.md`
  entry, a note like any other. A view costs nothing to abandon; a note
  survives being hand-edited; a record does neither.

  There is a third home, for the one thing the other two cannot serve: state
  only this app can read. "What changed since you last looked" needs a mark
  saying when that was, and no view can derive it. That mark goes in the
  browser — a `localmd:`-prefixed `localStorage` key or IndexedDB, under the
  storage contracts below — never in the folder, and it must be built to be
  lost. Losing it costs one prompt nobody sees; the same fact written into
  someone's folder is a record they did not ask for, cannot read and will not
  maintain, and a record left to a human starts to lie. The test is not "is it
  small". It is: **if this disappears, does anything the user owns become
  wrong?** If it does, it does not belong in the browser either — it belongs in
  a note.
- **Minimal and manual over clever and automatic.** Prefer a manual action + a
  native dialog over background automation; prefer deterministic tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whitefoxx/localmd](https://github.com/whitefoxx/localmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
