---
trigger: always_on
description: PROJECT-SPECIFICS LAYER — Dispatch.
---

<!--
  PROJECT-SPECIFICS LAYER — Dispatch.
  This is NOT the whole operating agreement. setup.sh assembles three layers, in order:
    1. universal core         (installed globally — the rigid rules: read-before-write,
                               prove-it, atomic commits, no secrets in tracked files,
                               evolve-the-harness)
    2. marketing-outreach     (what "done"/"verified" mean when a real human will receive
       profile                the email: approval-before-send, compliance, proven personalization,
                               never invent facts)
    3. THIS file              (the Dispatch-only sharpening below)
  When core and profile both speak, the stricter wins; the operator's explicit instructions win
  over both. Don't re-emit the core or the profile here — see README.md for how the layers stack.
-->

# Project: Dispatch

Dispatch is a weekly product newsletter plus light outreach for a small SaaS. Each issue is
drafted as Markdown, reviewed, then sent through an ESP (a Kit/Mailchimp-style service) that
tracks open and click rates. There is **no app to run** — the "user-facing surface" is the email
that lands in a real person's inbox, so every guardrail here is about what reaches that inbox.
Operator: **Nadia Rossi** (growth marketer). Work is tracked in **Linear**.

## Stack & layout

- **Drafts** are Markdown under `drafts/` — one file per issue, e.g. `drafts/2026-06-25-launch.md`.
  Front-matter holds `subject`, `preheader`, `segment`, and the planned `send_date`.
- **Sent issues** move to `sends/` once broadcast, with the final approved copy and a one-line
  result note (audience count, who approved). `sends/` is the historical record — never edit a
  file there; it's what actually went out.
- **The ESP** is the send + tracking system. We talk to it through its MCP (list/segment ops,
  draft a broadcast, pull stats) — but we **create drafts only, never auto-send** (profile rule).
- **Voice guide** lives at `voice/brand-voice.md` (tone, audience, banned words). Read it before
  writing copy; write recurring positioning decisions back to it (claude-mem persists them across
  sessions so the voice doesn't drift week to week).
- **The subscriber export** (`data/subscribers.sample.csv`) is a tiny FICTIONAL fixture for
  testing merge tokens — a handful of fake rows, no real people. The real list lives **in the
  ESP**, never in this repo.
- **Secrets:** the ESP API key is read from the `ESP_API_KEY` environment variable. It is **never**
  written to any file in this repo — not a draft, not a script, not a `.env` that gets committed.
  Real subscriber data (names, emails, anything that identifies a person) is **PII** and likewise
  never committed. The secret-scan hook (`--with-hooks`) is the deterministic backstop.

## What "done" means here

The profile's flow (research → draft → self-check the quality gates → **human approves** → send)
and its meaning of "verified" apply as-is. Dispatch sharpens them to these concrete bars:

- **Every claim is verifiable.** No invented stat, metric, customer count, testimonial, or "trusted
  by." A number in the copy traces to a real source we can point at; if we can't, it comes out or
  Nadia is asked. Over-claiming torches trust faster than plain copy ever could.
- **Links + UTM are correct, and tested.** Every link is clicked and lands where intended — never a
  staging URL, never an expired offer. Each carries the right UTM tags (`utm_source`, `utm_medium`,
  `utm_campaign`) so the tracked clicks attribute to the right campaign. A naked redirect that
  breaks counts as a broken link.
- **A TEST send to yourself before any broadcast.** Before the real audience, send the issue to
  Nadia's own seed address and open it in the actual inbox client — desktop and mobile. "It looks
  right in the ESP editor" is **not** verified; the editor lies about rendering, dark mode, and
  merge tokens.
- **Personalization is proven to render.** Every `{{first_name}}`-style merge token is checked
  against a real test profile *and* against the missing-value fallback. An empty or literal
  `{{token}}` shipped to thousands is the classic embarrassment.
- **Unsubscribe + consent respected.** Every send carries a working unsubscribe link and a real
  postal sender identity. Suppressed, unsubscribed, and bounced addresses are excluded. An address
  whose consent we can't confirm does not get mailed.
- **No PII committed.** The real list stays in the ESP. Nothing that identifies a person lands in
  `drafts/`, `sends/`, a script, or a commit message.

## verify.conf phases

`scripts/verify.sh` reads `.harness/verify.conf` and runs these top-to-bottom; the **first failure
stops the run** so you fix the earliest break instead of chasing a cascade:

1. **`spell` — `cspell` over the draft** — spelling and obvious typos are mechanical and cheap to
   catch, so they run first. A typo in a subject line is the most-seen, least-forgivable mistake.
2. **`links` — `lychee` over every link in the draft** — dead, staging, or wrong links are the
   failure that *looks* fine in the editor and only bites in the recipient's inbox. We added this
   phase after a broken link shipped once (see Gotchas) — it is the deterministic guard behind the
   "links tested" bar above.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PromptPartner/agentsmith](https://github.com/PromptPartner/agentsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
