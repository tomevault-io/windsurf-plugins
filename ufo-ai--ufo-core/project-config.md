---
trigger: always_on
description: `spec.md` is the source of truth for design — the core/extension doctrine, fixed decisions, the
---

# ufo — agent guidelines

## Where to look first

`spec.md` is the source of truth for design — the core/extension doctrine, fixed decisions, the
workspace model, and non-goals. Read it before proposing structural changes.

**Core doctrine is the first review question:** if a capability can be an extension, it is not
core. Every addition to `core/` must name why extensions cannot express it.

**Every member action happens in chat.** Connecting an account, granting access, approving a
change — a member expresses it in natural conversation and the agent drives it (a tool it calls,
surfacing any link in its reply); never a slash-command, keyword, or bespoke end-user HTTP
endpoint. The only endpoints are the chat transport itself, authenticated **read projections** of
object, status, usage, and audit data (a surface page reads directly), **prepared intents** — a
surface form's one mutation path: the panel's structured intent is admitted as a turn the engine
dispatches verbatim to the typed object verb, so the turn IS the chat transport and the audit
record, the route only prepares and admits, and the panel reads back the typed result or refusal —
a **stateless display render** (a composer rasterizing a member's picked file to show them what
they are about to send: bytes in, a picture back, storing nothing and admitting no turn — a preview
is not an action, it grants nothing and changes nothing, so it never has to be a chat turn) — and
unavoidable third-party plumbing (e.g. an OAuth callback). The speaker gates the granting act;
subsequent use is the wire's job. (`ufoctl` CLI verbs are the operator surface — a different
audience, not member actions.)

- Study how established products solve the problem before designing a solution. Adopt their proven
  patterns and conventions rather than inventing an approach from scratch.

## One shape

**The code is exactly what it does, nothing else.** Anything that creates a second answer to "what
is this" — past form, future form, conditional form, alias, hypothesis — is dead weight on the next
reader.

**No transition language.** Every file reads as if designed this way from day one: no `legacy`,
`deprecated`, `formerly`, `for now`, `v1`/`v2` staging, `TODO`, migration notes. This repo has no
past; salvaged code arrives as if written here.

## Enforce, don't document

If a constraint can be made true by code — a required argument, a type, a test, a CI gate — encode
it there and delete the prose. A cross-cutting precondition (workspace scoping, credential access,
sandbox egress) is established once at the boundary and threaded down; the unsafe primitive stays
module-private behind a factory, backed by a gate. Extensions import only `ufo.sdk` — a CI
gate forbids `core` internals in `extensions/`.

## Succinctness

Generated text and code are as succinct as possible — specs, docs, commits, code. Keep every
hard-to-vary decision; cut the words around it. Prefer a table to prose. One example, not three.

## User-facing copy

**Spartan and factual.** Every word a member reads — terminal screens, emails, web pages, agent
replies — states what happened, what is true, or what to do next. No UFO metaphors (`beam`,
`transmit`, `signal`, `saucer`, `mothership`, `identification`, `unidentified`): the product is
named ufo and that is the whole joke. No greeting, reassurance, exclamation, or restatement of what
the member just did. A line the member cannot act on and did not ask for does not ship.

**Default typography, zero styling.** Copy makes no typographic statement: sentences take standard
capitalization and end punctuation (one ending in a URL, path, or address drops the period);
headings and labels are cased normally — Title Case or sentence case, one convention per surface —
never lowercase-as-aesthetic (`Join Waitlist`); placeholder values are neutral (`email@work.com`).
Commands, addresses, and header names render verbatim (`curl`, `gmail.com`, `x-ufo-session`) even
at sentence start. Letter-spacing spelled in spaces (`u f o`) and decorative punctuation do not
ship — visual character lives in CSS and the mark, never in the characters of the words.

**`∵` is the mark and stays.** It is drawn, not said — the metaphor ban governs words, and the
mark is exempt from the punctuation rule that governs them. The terminal client opens a conversation
under the whole logo the mark is taken from, drawn in characters and held there by a test; strip
decoration around it, never it.

## Prompts

A change to text a model reads — a prompt section, a tool description, injected context — ships
only ablated: run the relevant evals with and without the changed wording, and with any other
section covering the same ground, and keep only what the arms prove load-bearing. A prompt change
no eval can measure gets that eval first. The eval suites and the ablation runner live outside this
repo; a maintainer runs the eval arms before merge.

## Skills

Creating or editing a Skill requires reading and applying
[Designing, Refining, and Maintaining Agent Skills at Perplexity](https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity)
as a review gate:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ufo-ai/ufo-core](https://github.com/ufo-ai/ufo-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
